If you are upgrading from `mod_auth_external` to `mod_authnz_external` then it may help to start by reading the page describing the [Apache Authn/z Architecture](AuthNZ.md).

After that:
<ol>
<li>
Make sure <code>mod_auth_external</code> is no longer being loaded.  You cannot load both <code>mod_auth_external</code> and <code>mod_authnz_external</code> without problems.  This means ensuring that there is no "<code>LoadModule</code>" or "<code>AddModule</code>" line for <code>mod_auth_external</code>.  You could also remove the <code>mod_auth_external.so</code> file from the Apache 'modules' directory.<br>
</li>

<li>
Install <code>mod_authnz_external</code> as described on the <a href='Installation.md'>Installation</a> page.<br>
</li>

<li>
The server-level configuration directives in the <code>httpd.conf</code> file can be left unchanged.  The "<code>AddExternalAuth</code>", "<code>AddExternalGroup</code>", "<code>SetExternalAuthMethod</code>", and "<code>SetExternalGroupMethod</code>" commands still work work the same way as before.  There was, however, a new, more compact alternate syntax was introduced in version 3.2.0 which can be used instead.<br>
</li>

<li>
In the per-directory configurations (either in <code>.htaccess</code> files or in a <code>&lt;Directory&gt;</code> block in <code>httpd.conf</code>) need to include a new directive to tell <code>mod_auth_basic</code> to use <code>mod_authnz_external</code> for authentication.  For <code>mod_auth_external</code>, the per-directory configurations normally looked something this:<br>
<pre><code>AuthType Basic<br>
AuthName &lt;authname&gt;<br>
AuthExternal &lt;keyword&gt;<br>
require valid-user<br>
</code></pre>

For mod_authnz_external, you need to add the "<code>AuthBasicProvider</code>" directive.<br>
<pre><code>AuthType Basic<br>
AuthName &lt;authname&gt;<br>
AuthBasicProvider external<br>
AuthExternal &lt;keyword&gt;<br>
require valid-user<br>
</code></pre>

The directive "<code>AuthType Basic</code>" tells Apache that you want to use the <code>mod_auth_basic</code> module to do "basic authentiation".  The directive "<code>AuthBasicProvider external</code>" tells <code>mod_auth_basic</code> to use <code>mod_authnz_external</code> to check the correctness of passwords.<br>
<br>
Note that the "<code>AuthBasicProvider</code>" directive is only needed if you are using <code>mod_authnz_external</code> for password checking.  If you are using it only for group checking, then this is not needed.<br>
</li>

<li>
If you were using mod_auth_external in a non-authoritative mode, then your per-directory configuration probably included the directive:<br>
<pre><code>AuthExternalAuthoritative off<br>
</code></pre>

This command will no longer work.  Instead you should use one or both of the following commands:<br>
<pre><code>AuthBasicAuthoritative off<br>
GroupExternalAuthoritative off<br>
</code></pre>

The "<code>AuthBasicAuthoritativ</code>e" directive effects password checking, which is done through <code>mod_auth_basic</code>.<br>
<br>
The "<code>GroupExternalAuthoritative</code>" effects only group checking.  That is if you had both "<code>GroupExternal</code>" directive setting up an external program for group checking, and an "<code>AuthGroupFile</code>" directive setting up a group file, then it would control whether the first module to process a "<code>Require group admin</code>" directive was the only one to run, or whether each group checker was given a chance to decide if the user was in that group based on it's group database.<br>
</li>

<li>
If you were using multiple Require directives, the behavior may change under Apache 2.2.  Suppose you wanted to allow access to user "<code>pete</code>" and members of the group "<code>admins</code>".  You might have do:<br>
<pre><code>Require group admin<br>
Require user pete<br>
</code></pre>
Under Apache 2.0, both of these directives would have been checked by <code>mod_auth_external</code>, and it would have correctly allowed access if either of the two conditions were satisfied.  In Apache 2.2, however, only "<code>Require group</code>" and "<code>Require file-group</code>" directives are checked by <code>mod_authnz_external</code>.  "<code>Require user</code>" and "<code>Require valid-user</code>" are checked by <code>mod_authz_user</code>, a standard module that comes with Apache.  How the two directives interact depends on whether they are authoritative or not.  <code>mod_authz_user</code> is Authoritative by default, so to get the old behavior, you will need to do<br>
<pre><code>GroupUserAuthoritative off<br>
</code></pre>
</li>

<li>
Note that a new type of functionality is available under Apache 2.2 with <code>mod_authnz_external</code>.  Thanks to <code>mod_authz_owner</code>, you can now do:<br>
<pre><code>Require file-owner<br>
</code></pre>
or<br>
<pre><code>Require file-group<br>
</code></pre>

The first checks if the name of the authenticated user matches the name of the unix account that owns the file.  The second checks if, according to whatever group database has been configured for the current directory, the currently authenticated user is in a group with the same name as the Unix group that owns the file.<br>
<br>
Normally these are rather strange directives, because normally unix accounts have no relationship to accounts in whatever database is being used for http authentication, but for people using '<code>pwauth</code>' with <code>mod_authnz_external</code>, these really check if the user has been authenticated as the unix user who owns the file.<br>
</li>
</ol>