mod\_auth\_external has moved (and Jan Wolter has died and is no longer maintaining it).  The new location is here https://github.com/phokz/mod-auth-external

## mod\_authnz\_external and mod\_auth\_external ##

**Previous Maintainers: [Jan Wolter](http://unixpapa.com), Tyler Allison (allison@nas.nasa.gov)**<br>
<b>Original Author</b>: <a href='http://www.unixtools.org/~nneul/'>Nathan Neulinger</a> (nneul@umr.edu)<br>
<br>
<br>
<code>Mod_authnz_external</code> and <code>mod_auth_external</code> are flexible tools for building custom basic authentication systems for the <a href='http://httpd.apache.org'>Apache HTTP Daemon</a>.  "Basic Authentication" is a type of authentication built into the HTTP protocol, in which the browser automatically pops up a login box when the user requests a protected resource, and the login ids and passwords entered are checked by Apache. <code>Mod_auth*_external</code> allows the password checking normally done inside Apache to be done by an separate external program running outside of Apache.  This is useful in either of two situations:<br>
<br>
<ul><li><b>Rapid, Safe Deployment of Custom Authentication Systems.</b>  Standard authentication modules with names like <code>mod_auth_file</code> and <code>mod_auth_ldap</code> exist for most common forms of password database, but occasionally you will need to authenticate out of some database for which no appropriate module exists.  Writing custom authentication modules is difficult, requiring a lot of knowledge of the internals of Apache, and bugs in such modules can crash Apache.  But with <code>mod_auth*_external</code> the custom code can be in a separate program, possibly even a Perl or PHP script.  The interface is very simple, and bugs in the authenticator program can not possibly crash Apache.</li></ul>

<ul><li><b>Authentication out of Secure Databases.</b> It is often undesirable for a password database to be readable by Apache.  If it is readable by Apache, then it is possible that bugs in Apache or in any CGI program run by Apache could allow hackers to access the password database.  With <code>mod_auth*_external</code> the external authenticator can be configured as a <a href='http://en.wikipedia.org/wiki/Setuid'>setuid</a> program, so that it runs as a different user than Apache, and so can access databases that are not accessible to Apache. Since only the small, simple authenticator program has the privileges to access the database, instead of all of Apache, this is vastly easier to make secure.</li></ul>

One of the most common secure databases that people want to authenticate out of is the Unix system password database.  The open source <a href='http://code.google.com/p/pwauth/'>pwauth</a> program is a <code>mod_auth*_external</code> compatible authenticator that can do this.  It can also authenticate from any <a href='http://en.wikipedia.org/wiki/Pluggable_Authentication_Modules'>PAM</a> authentication source.<br>
<br>
The obvious disadvantage of using <code>mod_auth*_external</code> is that each authentication requires that the authentication program be loaded and launched.  This causes some extra computational overhead.  Some hooks have been inserted into <code>mod_auth*_external</code> to make it easy to replace the call to an external authenticators with a call to a hardcoded internal authentication subroutine that you write.  This is sort of a half-way measure to just writing your own Apache module from scratch, allowing you to easily borrow some of the logic from <code>mod_auth*_external</code>, but you clearly lose the advantages of external authentication listed above.<br>
<br>
<code>Mod_auth*_external</code> can also be used to run external programs to make access control checks.  Access control means checking if a user is in a group allowed to access a particular resource.  It occurs after a user has been authenticated, by <code>mod_auth*_external</code> or by another module.<br>
<br>
<h2>Compatibility</h2>

<h3>Apache Versions</h3>

<table><thead><th> Apache 1.3 </th><th> mod_auth_external 2.1.x </th></thead><tbody>
<tr><td> Apache 2.0 </td><td> mod_auth_external 2.2.x </td></tr>
<tr><td> Apache 2.2 </td><td> mod_authnz_external 3.1.x or 3.2.x </td></tr>
<tr><td> Apache 2.4 </td><td> mod_authnz_external 3.3.x </td></tr></tbody></table>

The addition of "<code>nz</code>" to the module name in recent releases reflects the fact that the module has been redesigned to fit into the <a href='AuthNZ.md'>new authentication architecture</a> introduced by Apache, in which top level authentication modules named <code>mod_auth_basic</code> and <code>mod_auth_digest</code> call lower level modules with names like <code>mod_authn_file</code> and <code>mod_authn_dbm</code>.<br>
<br>
<h3>Windows, OS2, Netware, etc</h3>
Version 3.2.0 of <code>mod_authnz_external</code> was redesigned to avoid all unix system calls and work entirely through the Apache API.  In theory it should now work on any operating system supported by Apache, including Windows.  However, I do not know that anyone has tried this.  If you experiment with this, please let us know the results.<br>
<br>
<h3>Digest Authentication</h3>
<code>Mod_authnz_external</code> does <b>not</b> work with digest authentication.  It is unlikely that anyone would actually want to do this.  In digest authentication, the password is one-way encrypted before it is sent by the browser to the http server. It is only possible to check the validity of that password, if the password database contains either plain text passwords or passwords encrypted by exactly the method defined in the digest authentication standard.  If the database used some other one-way encryption method, then there would be no way to tell whether or not the password sent from the browser and the one in the database matched. So digest authentication could not be used with most reasonable authentication databases (storing plain text passwords is <i>not</i> reasonable). Digest authentication out of a Unix password database is impossible, for example.<br>
<br>
<h2>Security Considerations</h2>
Older versions of <code>mod_auth_external</code> would by default pass logins and passwords into the authentication module using environment variables.  This is insecure on some versions of Unix where the contents of environment variables are visible on a 'ps -e' command.  In more recent versions, the default is to use a pipe to pass sensitive data.  This is secure on all versions of Unix, and is recommended in all installations.<br>
<br>
People using <code>mod_auth*_external</code> with pwauth to authenticate from system password databases should be aware of the innate <a href='http://code.google.com/p/pwauth/wiki/Risks'>security risks</a> involved in doing this.<br>
<br>
<h2>Wiki Pages</h2>
<ul><li><a href='AuthList.md'>List of Available Authenticators</a>
</li><li><a href='HistoryNotes.md'>Historical License and Version Notes</a>
</li><li><a href='AuthNZ.md'>A Brief Explanation of the Apache Authn/z Architecture</a>
</li><li><a href='Links.md'>Links to Related Software</a>
</li><li><a href='FutureFeatures.md'>Ideas for Future Improvements to Mod_authnz_external</a></li></ul>

<ul><li><a href='Installation.md'>How to Install the Module into Apache</a>
</li><li><a href='Configuration.md'>How to Configure External Authenticators and Group Checkers</a>
</li><li><a href='AuthHowTo.md'>How to Write an External Authenticator or Group Checker</a>
</li><li><a href='HardCode.md'>How to Write an Internal Authentication Function</a>
</li><li><a href='UpgradeToNZ.md'>Upgrading from Mod_auth_external to Mod_authnz_external</a></li></ul>

<ul><li><a href='ModAuthzUnixGroup.md'>Unix Group Checking with the Mod_authz_unixgroup Module</a>