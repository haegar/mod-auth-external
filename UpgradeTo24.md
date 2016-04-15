If you are upgrading from Apache 2.2 to Apache 2.3 or 2.4 you will need to be aware of several changes in the configuration and behavior of mod\_authnz\_external.

First, you must use `mod_authnz_external` version 3.3.  Previous versions will not work with Apache 2.3 or 2.4.

### "`Require`" Command Syntax Changes ###

When using an external group checker, the syntax in Apache 2.2 and before was:

```
  GroupExternal <groupkeyword>
  Require group <groupname1> <groupname2> ...
```

In Apache 2.4, we need to replace the keyword `group` on the `Require` command with the keyword `external-group`, like this:

```
  GroupExternal <groupkeyword>
  Require external-group <groupname1> <groupname2> ...
```

Similarly, if you wanted to check if users were in the group that owns a file, the Apache 2.2 directive was:
```
  Require file-group
```
while in Apache 2.4 you must say:
```
  Require external-file-group
```

### Per Configuration Authentication versus Per URI Authentication ###

In Apache an HTTP request for a URI may sometimes trigger other "sub-requests" for other URIs. For example, there might be an internal redirect to a different page or a .shtml file might use a server-side include to fetch other files. Mod\_perl programs and the like can generate sub-requests very freely.

In previous versions of Apache, the authentication checks were redone each time a sub-request accessed a new URI.  In version 2.4, authentication modules repeat authentication checks for sub-requests only when it is for a resource with a different authentication configuration. So if you are doing a server-side include of a file that is in a directory where it has exactly the same authentication settings as the original, then the authentication check will only be done once.  This is good.

It might create a possible problem for a rare few users of mod\_auth\_external.  One of the environment variables passed to external authenticator programs is "URI". To quote myself:

> The URI variable is there because various people want it. Mostly it is useful
> not for authentication ("who is this person?") but for access control ("is this
> person permitted to do this?"), and good design usually dictates separating
> those functions. Strictly speaking, an authenticator is not the right place
> to be doing access control. However, mod\_authnz\_external is 50% a
> kludge-builder's tool, so we won't fuss if you want to break the rules.

Well, if broke the rule and wrote an external authenticator that uses the URI environment variable in a way that would lead it to giving different results for different URIs (not just for, say logging), and if you are doing anything at all that generates sub-requests, then you might be a bit messed up.

I think the number of people likely to be troubled by this issue is somewhere between "a couple" and "none". If you are among those people, you should either (1) redesign your authentication setup, or (2) edit the mod\_authnz\_external source code before installing it, changing three instances of "AP\_AUTH\_INTERNAL\_PER\_CONF" to "AP\_AUTH\_INTERNAL\_PER\_URI". I considered adding a configuration directive to mod\_authnz\_external that would let you switch the setting in a nicer way, but I suspect it would have to bypass the Apache API to work, and it's just not worth it.

### Authoritativeness ###

In Apache 2.4 the moderation between multiple different access control modules is now controlled entirely by mod\_auth\_basic, not by the individual modules. This means that the old mod\_auth\_external directives,

```
   GroupExternalAuthoritative
   AuthzExternalAuthoritative
```

no longer exist.

### Modifying Error Codes for Group Checking ###

In previous versions of mod\_authnz\_external, there was a directive which could
be used to send a different error code when an access control check failed:

```
   GroupExternalError 403
```

This also no longer exists.  The approximate same effect can be achieved with a new Apache directive:

```
   AuthzSendForbiddenOnFailure On
```

### Use With Mod\_authn\_socache ###

Apache 2.4 introduces an authentication caching module that can substantially improve the performance of mod\_authnz\_external by reducing the number of times the external authenticator needs to be run. For most users of mod\_authnz\_external, however, it also has some negative security implications as well. Please see the ConfigApache24 page for details.