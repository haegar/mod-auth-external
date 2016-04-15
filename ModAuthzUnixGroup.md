# Mod\_authz\_unixgroup #

**Author and Maintainer:** Jan Wolter [email](http://www.unixpapa.com/white.cgi)<br>
<b>Mailing List:</b> <a href='http://groups.google.com/group/mod_auth_external'>mod_auth_external@googlegroups.com</a>

<h2>Introduction:</h2>

<code>Mod_Authz_Unixgroup</code> is a unix group access control module for Apache 2.1 and later. If you are having users authenticate with real Unix login ID over the net, using something like my <a href='http://code.google.com/p/mod-auth-external'>mod_authnz_external</a> / <a href='http://code.google.com/p/pwauth'>pwauth\</a> combination, and you want to do access control based on unix group membership, then <code>mod_authz_unixgroup</code> is exactly what you need.<br>
<br>
There are different versions of mod_authz_unixgroup for different Apache releases:<br>
<br>
<table><thead><th> Apache 2.2 </th><th> Mod_authz_unixgroup 1.0.x </th></thead><tbody>
<tr><td> Apache 2.4 </td><td> Mod_authz_unixgroup 1.1.x </td></tr></tbody></table>

The configuration commands for these two versions are quite different.<br>
<br>
Let's say you are doing unix passwd file authentication with <code>mod_authnz_external</code> and <code>pwauth</code>. Your <code>.htaccess</code> file for a protected directory would probably start with the following directives:<br>
<pre><code>AuthType Basic<br>
AuthName mysite<br>
AuthBasicProvider external<br>
AuthExternal pwauth<br>
</code></pre>
That would cause <code>mod_auth_basic</code> and <code>mod_authnz_external</code> to do authentication based on the Unix <code>passwd</code> database. <code>Mod_Authz_Unixgroup</code> would come into play if you wanted to further restrict access to specific Unix groups. You might append the following directives:<br>
<br>
<b>Apache 2.2:</b>
<pre><code>AuthzUnixgroup on<br>
Require group staff admin<br>
</code></pre>
<b>Apache 2.4:</b>
<pre><code>Require unix-group staff admin<br>
</code></pre>
This would allow only access to accounts in the '<code>staff</code>' or '<code>admin</code>' unix groups. You can alternately specify groups by their gid numbers instead of their names.<br>
<br>
Though it makes the most sense to use <code>mod_authz_unixgroup</code> with unix passwd authentication, it can be used with other databases. In that case it would grant access if, (1) the name the user authenticated with exactly matched the name of a real unix account on the server, and (2) that real unix account was in one of the required groups. However, I think this would be a pretty senseless way to use this module. I expect that it will really only be used by users of <code>mod_authnz_external</code> and <code>pwauth</code> or other similar software.<br>
<br>
Some authentication modules, like <code>mod_auth_kerb</code>, use usernames that have domains appended to them, like "<code>whomever@krb.ncsu.edu</code>". In such cases, <code>mod_authz_unixgroup</code> will take the part before the <code>@</code>-sign as the username and ignore the rest.<br>
<br>
It will come as no surprise that this module works only on Unix systems. It should work on pretty much any vaguely modern Unix.<br>
<br>
<h2>Installation and Configuration</h2>

To install the module in Apache, you follow pretty much the <a href='Installation.md'>same procedure</a> as for <code>mod_auth_external</code>.<br>
<br>
The configuration commands for mod_authz_unixgroup are entirely different depending on whether you are using Apache 2.2 or Apache 2.4.  Choose your poison:<br>
<ul><li><a href='ModAuthzUnixGroup22.md'>Apache 2.2 Configuration</a>
</li><li><a href='ModAuthzUnixGroup24.md'>Apache 2.4 Configuration</a>