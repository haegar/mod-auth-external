## Notes: ##

Before installing, note the following:

  * These instructions are for the current version of `mod_authnz_external`.  The installation procedure for older versions is very similar, and is documented in the INSTALLATION file included in the distribution.

  * Make sure that you are using the correct version of `mod_auth_external` for your version of Apache:
| Apache 2.4 | mod\_authnz\_external-3.3.x |
|:-----------|:----------------------------|
| Apache 2.2 | mod\_authnz\_external-3.1.x or mod\_authnz\_external-3.2.x |
| Apache 2.0 | mod\_auth\_external-2.2.x   |
| Apache 1.3 | mod\_auth\_external-2.1.x   |

> You can check your apache version by running it from the command line with the -v flag.

  * Originally, `mod_auth_external` was a stand-alone module.  However a new authentication module structure was introduced in Apache-2.1, where `mod_auth_basic` and `mod_auth_digest` are the only top-level authentication modules.  All other authentication modules simply provide authentication services to these modules, and have names starting with "`mod_authn_`" for authentication modules, or "`mod_authz_`" for access control modules, or "`mod_authnz_`" for modules that provide both services.  `Mod_authnz_external` is designed to fit into this new structure.  It has essentially the same features as `mod_auth_external`, but there are differences in the configuration commands.

> It should be noted that it is still possible to use older-style independent authentication modules in Apache 2.2, and `mod_auth_external-2.2.x` can be made to work with only a little difficulty arising from `mod_auth_basic`'s reluctance to be turned off.  See the `mod_auth_external INSTALL` document for information on using it with Apache 2.2.    Do not, however, install both mod\_auth\_external and mod\_authnz\_external in your httpd.  I don't know what exactly would happen, but it won't be good.

  * If you are upgrading from `mod_auth_external` to `mod_authnz_external` read the [Upgrading to Mod\_authnz\_external](UpgradeToNZ.md) page.

  * If you want to use `mod_authnz_external` with a hard-coded internal authentication function rather than with an external authenticator, then follow the [Hard Code Installation instructions](HardCode.md) before follow the instructions on this page.

  * Starting with version 3.2.x, mod\_authnz\_external is designed to work on any platform supported by Apache.  Previous versions were Unix-only.  So mod\_authnz\_external might work on Windows, but the author doesn't really do Windows development and doesn't even own a Windows C compiler.  So it has not been tested at all, no pre-compiled Windows code is available, and there are no installation instructions for non-Unix platforms.  If you figure any of this out, please consider contributing your findings.

  * There are two ways of installing mod\_authnz\_external on a Unix system.
    1. You can statically link it with Apache.  This requires rebuilding Apache in such a way that 'mod\_authnz\_external' will be compiled in.  This is rarely done in modern Apache installations, but might be done if you want to get slightly better performance.
    1. You can make mod\_authnz\_external a dynamically loaded module.  If your Apache has been built to support dynamically loaded modules you can do this without rebuilding Apache, so it is pretty easy  For information on dynamically loaded modules see http://www.apache.org/docs/dso.html .

> This page gives instructions only for installing Apache as a dynamically loaded module.  There are instructions for installing it statically in the INSTALLATION file in the distribution, but it has been so long since I have done this, that I do not know if they are still correct.

## Step-by-Step Directions ##

### (1) Check if Dynamically Loaded Modules are Supported ###

Ensure that your Apache server is configured to handle dynamically loaded modules.  These days, nearly all are, but if you want to check this, run Apache server with the -l command flag, like
```
      httpd -l
```
If 'mod\_so.c' is one of the compiled-in modules, then you are ready to go.  Note that some installations may give the http daemon different names, like 'apache' or 'httpd2'.  Some may have multiple copies of apache sitting in different directories.  Be sure you looking at the one that is being run.

### (2) Compile the Module ###

Use the following command in the 'mod\_authnz\_external` distribution directory:
```
     apxs -c mod_authnz_external.c
```
'Apxs' is the Apache extension tool.  It is part of the standard Apache distribution.  If you don't have it, then there may be a Apache development package that needs to be installed on your system, or your Apache server may not be set up for handling dynamically loaded modules.  Some systems rename it weirdly, like 'apxs2' in some openSUSE distributions.

Apxs should create a file named '`mod_authnz_external.so`'.

_AIX USERS_:  If you have problems at this point, see the notes in the INSTALL file.

### (3) Install the Module ###

Apxs can do this for you too.  Do the following command (as root so you can write to Apache's directories and config files):
```
       apxs -i -a mod_authnz_external.la
```
This will create `mod_authnz_external.so` and copy it into the proper place, and add appropriate `AddModule` and `LoadModule` commands to the configuration files.

### (4) Configure the Module ###

The next step is to do the necessarily Apache configuration. The procedure is substantially different in different versions of Apache. See whichever of the pages below that applies you your Apache version for details:

  * ConfigApache22 - Apache version 2.2
  * ConfigApache24 - Apache version 2.3 or 2.4

### (5) Install the Authenticator ###

Install your external authentication program in the location named by the pathname in the `AddExternalAuth` directive that you configured.

Make sure everything is permitted so that whatever account the `httpd` runs under can execute the authenticator.  Typically this requires 'execute' access to the script and all the directories above it.  If it is a script, then read access to the script will also be needed.

If your script is an set-uid script, then make sure the file is owned by the user it is supposed to run as, and that the suid-bit is set.

### (6) Restart Apache ###

Restart Apache, so that all the new configuration commands will be loaded.  If you have the `apachectl` command do:
```
apachectl restart
```
For some systems which doesn't have `apachectl`, you'll want to manually run the startup script for apache.  The locations of these vary somewhat in different Unix systems, but they typically are something like this:
```
/etc/init.d/httpd restart
```

### (7) Test It ###

Test your changes/code by trying to view a protected page.

If it doesn't work, check the apache error logs.  They are loaded with helpful information.  Some common problems and their usual causes:

  * Miscellaneous odd behaviors.

> Did you restart the httpd after the last time you edited the  httpd.conf file, installed a new version of the module, or recompiled Apache?  Confirm that an  "Apache configured -- resuming normal operations" message appeared  in the error log when you restarted.

  * Apache complains about not recognizing `mod_authnz_external` commands in the `httpd.conf` file like "`DefineExternalAuth`" and "`AddExternalAuth`".

> Either the module didn't get installed (if you staticly linked the module, are you running the newly compiled copy of `httpd`?), or it isn't enabled (if it is dynamically linked, the `AddModule` and `LoadModule` commands described on the configuration pages may be missing, incorrect, or commented out by an inappropriate `<IfDefine>`).

> Sometimes I've found that the `httpd.conf` file I've been editing is not actually the one being used by the copy of Apache that is running.  Sometimes I test this by inserting deliberately invalid commands and checking to see if error messages are generated when Apache is restarted.

  * It displays pages in a protected directory without asking for a login and password.

> For some reason Apache is not seeing the directory configuration commands that set up authentication for that directory.  If you are using `.htaccess` files, does your `httpd.conf` file say "`AllowOverride AuthConfig`" for the directory?  Apache is usually distributed with "`AllowOverride None`" set, which will cause `.htaccess` files to be quietly ignored

  * All logins are rejected, and the error log says it cannot execute the authentication module.  Error messages might look like:
```
exec of '/foo/bar/authcheck' failed: (2) No such file or directory
[Thu Nov 15 12:26:43 2007] [error] AuthExtern authcheck
   [/foo/bar/authcheck]: Failed (-1) for user foo
[Thu Nov 15 12:26:43 2007] [error] user foo: authentication
   failure for /mae/index.html": Password Mismatch
```

> The first of these three messages is from Apache's process launching library, and gives the clearest information about what caused the error.  Typically it will be either "No such file", which means that the pathname you specified for the authenticator in the Apache configuration file does not match the actual location of your external authenticator, or it will be "permission denied", indicating that either the file or one of the directories above it is permitted so whatever account apache is configured to run as does not have execute permission. If it's a script, it also needs read permission.

> The second error message is actually generated by mod\_auth\_external.  It just says authentication failed for the user.  Normally it would give the status code returned by the authenticator in parenthesis, but if the authenticator could not be executed it will show a phoney status code of -1 (which some systems display as 255).

> The third error message is from Apache.  Don't be mislead by it's saying "Password Mismatch".  When `mod_auth_external` fails, it rejects all access attempts.  To apache this looks like a password mismatch.

  * Authentications failed and the message in the error log says it failed with a status code of -2 or 254, for example:
```
[Thu May 21 12:29:46 2009] [error] [client 127.0.0.1]
   External authenticator died on signal 9
[Thu May 21 12:29:46 2009] [error] [client 127.0.0.1]
   AuthExtern authcheck [/foo/bar/authcheck]: Failed (-2) for user foo
[Thu May 21 12:29:46 2009] [error] [client 127.0.0.1]
   user foo: authentication failure for "/mae/index.html": Password Mismatch
```

> The status code of -2 (or 254) in the second message indicates that the authenticator crashed or was killed before it could return a status code.  The first message identifies the particular signal number that terminated the process, either because some other process sent the signal, or because the program executed an illegal command causing a segmentation fault.  Again, the "Password Mismatch" message from Apache is pretty meaningless.

  * There are several other internal errors that can theoretically occur in mod\_authnz\_external which will also be reported with error messages and negative status codes, similar to the previous example.  These are rare and generally indicate really unusual exhaustions of system resources or bugs in mod\_authnz\_external, apache, or other modules.  Read the source or contact the mod\_authnz\_external developers for help.

  * Error log says "Failed (X) for user foo" with X being some usually positive number and no other more specific error messages.

> This means the authenticator ran, and exited with the given non-zero return code.  You'll have to check the authenticator to see under what conditions it exits with that return code. This is the normal expected behavior for mod\_authnz\_external when an authenticator rejects a login.