`Mod_auth_external` is, of course, useless unless it has an external authenticator to run. There are some examples of authenticators included in the distribution, and several useful ones are available separately.

## Example External Authenticators in Distribution ##

<dl>
<dt><code>test</code></dt>
<dd>
Several small dummy external authentication programs written in Perl. These are meant only for testing of <code>mod_auth*_external</code>. They accept any user whose password exactly matches his login name. They write lots of debugging info to the error_log file.<br>
<br>
<b>Author and Maintainer</b>: <a href='http://www.unixpapa.com/white.cgi'>Jan Wolter</a>

</dd>

<dt><code>auth-mysql</code></dt>
<dd>
A external authenticator for authenticating out of a MySQL database. Written in Perl using the DBI/DBD library, so it is easily adapted to any other SQL server. (These days, using Apache's mod_authn_dbd is probably a better choice for most appications.)<br>
<br>
<b>Author and Maintainer</b>: <a href='http://anders.fix.no/software/#unix'>Anders Nordby</a>

</dd>
</dl>

## Example Internal Authenticators in Distribution ##

<dl>
<dt><code>radius</code></dt>
<dd>
A Radius client using code from the publicly available Merit Radius source code. <b>Unmaintained.</b>

<b>Author</b>: Tyler Allison (allison@nas.nasa.gov)<br>
<br>
</dd>

<dt><code>sybase</code></dt>
<dd>
A function that queries a sybase database and compares the passwords for said user.<br>
<b>Unmaintained.</b>

<b>Author</b>: br@ota.fr.socgen.com<br>
</dd>
</dl>

## Authentication Modules Available Separately ##

`Mod_auth*_external` can be used either with authenticators specifically written for it or with [checkpassword](http://cr.yp.to/checkpwd.html) authenticators.  This is not a [complete list](http://www.qmailwiki.org/index.php/Qmail-checkpassword) of `checkpassword` authenticators, but I'll include a few here.

<dl>
<dt><code>pwauth</code></dt>
<dd>
A setuid-root external authentication program for securely authenticating out of most flavors of Unix shadow password files, or via PAM. Combined with 'pam_smb', this supports NT-style SMB authentication. Supports some Unix lastlog and faillog options.<br>
<br>
<b>Available From:</b> <a href='http://code.google.com/p/pwauth/'>http://code.google.com/p/pwauth/</a>

<b>Author and Maintainer</b>: <a href='http://www.unixpapa.com/white.cgi'>Jan Wolter</a>
</dd>

<dt><code>vcheck</code></dt>
<dd>
An external authenticator by Anders Brander for use against a vpopmail user database.<br>
<br>
<b>Available From:</b> <a href='http://anders.brander.dk/stuff/workings/vcheck/'>http://anders.brander.dk/stuff/workings/vcheck/</a>

<b>Author and Maintainer</b>: Anders Brander<br>
</dd>

<dt><code>checkpassword</code></dt>
<dd>
The original checkpassword program authenticates from the Unix system password file, like pwauth.<br>
<br>
<b>Available From:</b> <a href='http://cr.yp.to/checkpwd/install.html'>http://cr.yp.to/checkpwd/install.html</a>

<b>Author:</b> D. J. Bernstein<br>
</dd>

<dt><code>checkpassword-pam</code></dt>
<dd>
Another way to authenticate via PAM.<br>
<br>
<b>Available From:</b> <a href='http://checkpasswd-pam.sourceforge.net/'>http://checkpasswd-pam.sourceforge.net/</a>

<b>Author:</b> Alexey Mahotkin alexm@hsys.msk.ru<br>
</dd>

<dt><code>checkpassword-ldap</code></dt>
<dd>
An LDAP authenticator.<br>
<br>
<b>Available From:</b> <a href='http://freshmeat.net/projects/checkpassword-ldap/'>http://freshmeat.net/projects/checkpassword-ldap/</a>
</dd>

<dt><code>checkpasswd-imap</code></dt>
<dd>
An IMAP authenticator written in Python that caches credentials for better performance and so you don't overrun server rate limits.<br>
<br>
<b>Available From:</b> <a href='http://www.namazustudios.com/blog/checkpasswd-imap-a-mod_authnz_external-style-password-checker/'>http://www.namazustudios.com/blog/checkpasswd-imap-a-mod_authnz_external-style-password-checker/</a>

<b>Author:</b> Patrick Twohig<br>
</dd>

<dt><code>radcheckpassword</code></dt>
<dd>
Checking against external Radius server(s).<br>
<br>
<b>Available From:</b> <a href='http://free.acrconsulting.co.uk/email/radcpw.html'>http://free.acrconsulting.co.uk/email/radcpw.html</a>

<b>Author:</b> Andrew Richards<br>
</dd>
</dl>