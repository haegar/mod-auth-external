# Configuring Mod\_authz\_unixgroup on Apache 2.4 #

**NOTE: Configuration commands for Apache 2.2 and Apache 2.4 are quite different. Please make sure you are reading the correct instructions.**

To use mod\_authz\_unixgroup in a particular directory, you must call it up by including the 'unix-group' or 'unix-file-group' provider names on the 'Require' directive in the `.htaccess` file for the directory or an appropriate `<Directory>` block in `httpd.conf`.

```
Require unix-group admin
```
or
```
Require unix-group students teachers staff
```
The second version allows a person to have access to the directory if he is in any one of the listed groups.  You can identify groups using their group id numbers rather than their names, if you prefer.

Obviously this only makes sense in a directory where you are doing
authentication.  This could be any kind of authentication, but it makes
most sense if you are using it in combination with authentication out of
the unix password file.

A user is considered to be in a group if either (1) the group is the user's primary group identified by it's gid number in `/etc/passwd`, or (2) the group is listed in `/etc/group` and the user id is listed as a member of that group.

## Use with mod\_authz\_owner: ##

You can use `mod_authz_unixgroup` together with `mod_authz_owner` by doing:
```
Require unix-file-group
```
This would allow access to a file only if the user was a member of whichever unix group owns the file.

You may have to install `mod_authz_owner` before this will work.  Though it is part of the standard Apache distribution, it is not usually installed by default.

## Failure Handling ##

Note that when access is denied, either because the file does not exist or because the user is not in the group that owns the file, then the normal response will be for the browser to flush it's cached login and password for the authentication realm, and give the user a new login prompt. The user will have to re-login to access other files in the realm that they do have access to. This may be clumsy in some applications, where it would be much nicer to display a "permission denied" error message and not flush the user's credentials.  This can be achieved by using the standard Apache directive:
```
AuthzSendForbiddenOnFailure On
```
You may want to customize the 403 error page if you do this.