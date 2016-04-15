## License and Copyright ##

The first version of `mod_auth_external` was written by Nathan Neulinger.  My theory is that he started with some apache module, like 'mod\_auth\_file' and started editing it to turn it into `mod_auth_external`.  He just left the original Apache license in there. It was, after all, still partly Apache's code.  This is why the copyright message in 'mod\_auth\_external' says the copyright is held by the Apache Group, although the Apache group is in no way responsible for this code.

Since then `mod_auth_external` has suffered three  major rewrites and many small evolutionary changes, and about all that is untouched is the 1995 vintage Apache copyright/license message in front.  Does this make sense?  The authors of 'mod\_auth\_external' haven't the faintest idea and none of them care very much.

## Version Branches ##

There used to be another version of `mod_auth_external` maintained by Satoh Fumiyasu. I believe the last version was called version 3.0.0beta3. The versions seem to have diverged after version 2.0.0, Tyler Allison's next-to-last release. It differed significantly from this version, and was not precisely compatible. Fumiyasu's version supported authenticating through a socket against an authentication daemon, a feature I may support in the future. It's documentation was mostly in Japanese. This seems to have largely disappeared from sight.

However, this is the reason why the version of `mod_auth_external` distributed here skipped the version number 3.0.