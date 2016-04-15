Here are some relevant links.  See also the [list of available external authenticators](AuthList.md):

## Other Software ##

<ul>
<li><a href='http://httpd.apache.org/'>Apache HTTP Server</a>.  Can't run <code>mod_auth_anything</code> without it.<br>
</li>

<li><a href='http://code.google.com/p/pwauth/'>pwauth external authenticator</a>. Used with <code>mod_auth*_external</code> for authenticating out of a unix password database or via PAM.<br>
</li>

<li> <a href='http://sourceforge.net/projects/mod-auth-shadow/'>mod_auth_shadow</a>, by Bruce Duggan, appears to be a re-implementation of the same basic concept for authenticating out of shadow password file.  I haven't checked it out enough to decide if it has any advantages over using <code>pwauth</code> with <code>mod_auth_external</code>. On first inspection it appears to be slightly less portable and paranoid.<br>
</li>

<li><a href='http://linux.wareseeker.com/Internet/mod-auth-pipe-1.0.zip/319312'>mod_auth_pipe</a> is Alvaro Gamez Machado's development from <code>mod_auth_shadow</code>, allowing it to run arbitrary authenticators.  The result ends up doing pretty much the same thing as 'mod_auth_external'.<br>
</li>

<li><a href='http://mod-auth-script.sourceforge.net/'>mod_auth_script</a> looks like it can be used to perform similar functions to <code>mod_auth_external</code>.  It runs the authenticator by generating a sub-request to a CGI program.  I haven't analyzed this approach, but it has obvious advantages in that the authenticator programs are just normal CGIs.<br>
</li>

<li><a href='http://benow.ca/forum/News/Modification%20of%20mod-auth-external%20to%20enable%20authentication%20caching.'>Modification of mod-auth-external to enable authentication caching</a>. This is a branch of mod_authnz_external 3.2.5 modified to do authentication caching in module. It caches the username and IP address only, so until the timeout, further requests with the same IP address and username will be automatically accepted without running the external authenticator. I find it worrisome that those authentication requests will be accepted even if the password is not the same. At least that has the upside of not creating a less secure replica of your password database like mod_authn_socache sometimes does, but I can't say I'm very happy about it.  Note also that the cache is not in shared memory, so that each Apache process maintains its own cache, though it would still give a substantial performance boost. I'm inclined to think mod_authn_socache will be a better solution, when available, but I'm not entirely happy with either.<br>
</li>

<li><a href='http://www.kernel.org/pub/linux/libs/pam/'>PAM</a>. Portable Authentication Modules are libraries that have a common interface and can be linked to a program to do authentication out of different databases.  Linux, FreeBSD and Solaris support PAM.  OpenBSD does not.<br>
<br>
If you want to authenticate from a PAM module, but the user your httpd runs as does not have the necessary access, then the <code>pwauth</code> external authenticator can be run from <code>mod_auth*_external</code> to do the PAM authentication.<br>
<br>
The <a href='http://www.kernel.org/pub/linux/libs/pam/modules.html'>list of PAM modules</a> includes authenticators for Kerberos, radius, unix password or shadow files, SMB, various SQL databases, and just about anything else imaginable.<br>
</li>

<li><a href='http://pam.sourceforge.net/mod_auth_pam/'>mod_auth_pam</a>. If you want to use a PAM module to authenticate, and whatever user Apache runs as has the necessary access to do the check, then you don't need an external authenticator, and you should probably use this module instead of <code>mod_auth*_external</code> and <code>pwauth</code>.  There is such a thing as mod_auth_pam2, which is supposed to work with Apache2.<br>
</li>

<li><a href='http://www.carc.musc.edu/webNIS/mod_auth_any.html'>mod_auth_any</a>. This seems to be similar in function to <code>mod_auth*_external</code>. It seems very sparsely documented at this stage, but it from  looking at the source code it seems to pass the login/password to the external authenticator on the command line, which doesn't seem very secure since they'd be trivially visible to anyone doing a 'ps'.<br>
</li>

<li><a href='http://modules.apache.org/'>Apache Module Registry</a>. A good place to find apache modules.<br>
</li>

<li><a href='http://www.namazustudios.com/blog/checkpasswd-imap-a-mod_authnz_external-style-password-checker/'>checkpassword-imap</a>. An IMAP authenticator written in Python for mod_auth_external that caches credentials for faster performance and so that you don't run up against rate limits on the IMAP server.<br>
</li>

<li><a href='http://www.compassnetworks.com.au/?page=mod-auth-external'>BlueQuartz Authentication Script</a> seems to be an external authentication script for PAM written in Perl, similar to pwauth.  Seems to have some good security features, but I wasn't able to figure out how to get the source code.<br>
</li>
</ul>

## How To Pages ##

<ul>

<li>
Jonathan Weiss's blog entry describing <a href='http://blog.innerewut.de/2007/6/26/apache-2-2-authentication-with-mod_authnz_external'>how to install mod_authnz_external and pwauth</a> on FreeBSD. Might be a useful reference.<br>
</li>

<li>Gatis Špats describes using mod_auth_external 2.2 to authenticate from a MS SQL 2000 server and gives source code for a Javascript authenticator on <a href='http://www.howtoforge.com/apache2_authentication_mssql2000'>HowToForge</a>.  The installation instructions are a bit obsolete, but the Java authenticator would likely work as well as ever.<br>
</li>

<li>Someone posted <code>mod_authnz_external</code> and <code>pwauth</code> <a href='http://en.gentoo-wiki.com/wiki/Apache2/mod_authnz_external'>installation instructions on the Gentoo Wiki</a>.  This includes an incorrect statement that there is a bug in <code>pwauth</code> which can be fixed by changing a <code>getuid()</code> call to a <code>geteuid()</code> call.  Making that change actually disables one of <code>pwauth</code>'s security features and should not be done.<br>
</li>

<li>Instructions on <a href='http://moinmo.in/HowTo/Setup'>Moin with WSGI Apache HTTPAuth how to set up the MoinMoin Wiki Engine with mod_authnz_external</a> including a sample authenticator writting in ruby.<br>
</li>

<li>Notes on <a href='http://www.d90.us/toolbox/2008/11/03/configuring-authnz_external-and-pwauth/'>configuring authnz_external and pwauth</a> which looks like it might be for Redhat Linux.<br>
</li>

</ul>