Several design changes are under consideration for `mod_authnz_external`.  I'm interested in feedback on these ideas, or in other ideas that people might find useful.

## Splitting the Module ##

It would be possible to split the module into two modules, `mod_authn_external` which would do password checking and `mod_authz_external` which would do group checking.  The original justification for combining the two was that there was a lot of shared code between them, but the 3.2 rewrite reduced that, because much of the business of launching the external process was farmed out to Apache's process management library.  At this point the main area of overlap is the `exec_external()` which is largish, but not so largish that the need for combining the two modules is compelling.

The main advantage would be in replacing one complex module with two simpler modules.  People who only need one of them, which I think means most people, would be installing a smaller module with simpler configuration instructions.  The minority who use both would have to install two modules, but configuration would still be much the same.

We'd have to split the `AuthExternalContext` command into two commands each effecting one module.

At this point I'm not quite ready to do this.  The benefits aren't that great, and it's confusing enough having `mod_authnz_external` and `mod_auth_external` without adding `mod_authn_external` and `mod_authz_external` to the mix.  Maybe if something else comes along to compel another major re-design of the module, then this could be done at the same time.

## Authentication Daemons ##

`Mod_authnz_external` currently works by running the authenticator each time an authentication is needed.  You might get better performance if the authenticator were a persistent daemon to which `mod_authnz_external` made socket connections each time an authentication was needed.  Probably we'd define the authenticator with a command like:
```
DefineExternalAuth <keyword> server <hostname>:<portnumber>
```
When an authentication was needed, `mod_authnz_external` would open a socket connection to the given host and port.  If `<hostname>` was "`localhost`" and we were running on a Unix system it would do a UNIX socket.  It would write the login name and password to the socket, and the authenticator would check them and write back either "OK" or an error message which `mod_authnz_external` would log.

There are obvious performance advantages to this.  Establishing a socket connection is probably faster than launching a new process.  Plus the authenticator can do things like maintaining persistent database connections and caching recent authentications that can further improve it's performance.  Plus it opens up the possibility of having the authenticator running on a different server than the HTTP daemon.

There are some down sides too though.  Certainly the authenticator is going to be much harder to write.  Instead of just reading from stdin and returning an exit code, it must listen for connections on a socket.  However we could supply sample templates for that part of the code and people could just fill in their own authentication code.

Authentication daemons could become a bottleneck.  There are usually many httpd processes running on a server.  If they all have to go to the same authentication daemon for their authentications, then this could be a problem on busy servers.  And if the server isn't busy, why use a daemon at all?

Problems could also arise if the authentication daemon died or stopped responding.  One possibility would be to have `mod_authnz_external` restart the daemon if it is not responding.  We could introduce another syntax, like this:
```
DefineExternalAuth <keyword> daemon <portnumber>:<pathname>
```
Unlike the "`server`" option, this would always have the daemon running on localhost, with the given port number.  If it fails to respond, then it would run the program at the given pathname, wait a few seconds, and try again to make a socket connection.

The daemon would have to be built with the usual restart logic, where upon start up it checks if a copy is already running, kills that if it exists, and then starts serving.  So again the complexity of the authenticator goes up.

None of this would be terribly hard to build into `mod_authnz_external`.  There are already hooks in the `exec_external()` function that would make it able to launch daemons.  I'd need to research to see what kind of socket management can be done through the Apache API (see [here](http://apr.apache.org/docs/apr/trunk/group__apr__network__io.html) and [here](http://dev.ariel-networks.com/apr/apr-tutorial/html/apr-tutorial-13.html)), since I'd prefer not to make this Unix dependent.  One of the biggest questions would be the protocol used to communicate over the socket to the authenticator.  Many protocols exist that include authentication.  Could we do something compatible with one of those that would allow us to use some existing daemons with this?

But I think that before I under took this I'd want to have someone who seriously wanted this feature and was willing to invest some time on testing and development.  I don't have a production server where this would make sense, and if I construct it as a pure theoretical exercise, then I'm afraid it might turn out unusable in practice.

Actually, I'm afraid of that in any case.  Good, stable, scalable server programs are hard to write, and without one an authentication system built on this would tend to be fragile.  I think the existing method of starting up a fresh authenticator for each request may well be better for most applications.

## Caching Authentications ##

If you have a user logged onto your system, then you need to re-authenticate him for every HTTP request he makes.  This is stupid.  It should be possible to cache the fact that a given login/password pair is valid, and skip launching the authenticator if we recently checked that pair.

Actually, this shouldn't be part of `mod_authnz_external` but should be a separate module. There is already a [mod\_auth\_cache](http://mod-auth-cache.sourceforge.net/) but it appears to cache the login data in a cookie, where it can readily be manipulated by the user.  This is completely unacceptable.  The cache needs to be on the server side.

The cache could simply be kept in Apache's memory, but there are typically many copies of Apache running, and ideally you'd like the cache to be shared across them all.  Possibly a dbm file or shared memory could be used.  If you were using a disk cache, you'd probably do something like take an MD5 checksum of login/password/domain, and use that as a key into the dbm file, with the last access time as the value.  With a stored memory solution you could probably skip the encryption.

There seems to be a [caching option for mod\_auth\_ldap](http://www.muquit.com/muquit/software/mod_auth_ldap/mod_auth_ldap_cache.html) that uses a dbm file.  Might be worth looking at that as an example.

Apache 2.4 will have a [ap\_socache.h](http://svn.apache.org/repos/asf/httpd/httpd/trunk/include/ap_socache.h) interface that defines a new distributed small-object cache interface. Should check if that is going to be useful.