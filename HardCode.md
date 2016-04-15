There are some hooks in `mod_authnz_external` designed to make it easy to replace the call to an external authenticator with a call to an internal function.  This is a bit weird, as such internal functions have none of the advantages of an external authenticator, but this can be a short-cut way to create a custom authentication module.

The procedure is as follows:

### Step 1: ###

Edit "`mod_authnz_external.c`"

### Step 2: ###

Uncomment the `_HARDCODE_` #define.

### Step 3: ###

Uncomment the line:
```
/* #include "your_function_here.c" */
```

Actually, I think it might be better to embed the function itself directly in this file instead of including it.  Modules work better if they are implemented in a single source file.

Your function should start something like:
```
int example(char *user_name,char *user_passwd,char *config_path)
```
Obviously you should call it something other than `example`, but the rest of the function declaration should be just as above.

It should return 0 if the password is correct for the given user, and other values if it is not, pretty much just like external authentication programs do.

You'll want to code this very carefully.  A crash will crash not just your program but the entire httpd.

**DO NOT** use exit() or other such calls that will cause your function to exit abnormally or dump core. It will take the entire httpd with it and display a message to your browser saying "no data".  Use "return" instead of exit().

### Step 4: ###

Choose an all-caps function name for your function. ie: 'RADIUS' or 'SYBASE' This will be used for telling mod\_authnz\_external which hard coded function you want to call.  In this documentation we will call it `EXAMPLE`.

### Step 5: ###

Find the `exec_hardcode()` function inside `mod_authnz_external.c`. Find the big commented section there.  Replace the example call to `example()` with a call to your function.  Also change the name "`EXAMPLE`" to the name you chose for your function.

The function call in `exec_hardcode()` should look something like:
```
if (strcmp(check_type,"EXAMPLE")==0) {
    code = example(c->user,sent_pw,config_file);
}
```

Here, we replace "EXAMPLE" with the name you chose in step 4 and "example" with whatever you called your function in step 3.

### Step 6: ###

Save your work.  Also save some whales.

### Step 7: ###

Compile and configure mod\_authnz\_external as described on the [Installation](Installation.md) and [Configuration](Configuration.md) pages.

The AddExternalAuth command in your httpd.conf file might look something like
```
AddExternalAuth whatever EXAMPLE:/usr/local/data/configfile
```
Here 'whatever' is the name you will use to invoke this authenticator in the AuthExternal commands in your .htaccess files.  'EXAMPLE' is the name you choose in step 4 and inserted into the "if" statement in step 5.  Any data after the colon will be passed into your function as the argument called "'config\_path`" in the sample function declaration in step 3. It might be a config file path or something else.