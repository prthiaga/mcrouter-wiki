### Version 36 release notes:

* [f473934](https://github.com/facebook/mcrouter/commit/f4739340892931968ede97f6d27dbec1ebbe2a28) fixes a security issue. Updating to a version that includes that fix (i.e. version 36 or greater) is highly recommended.
* New option in MissFailoverRoute allows returning the “best” reply instead of the last reply. Check this [wiki page](https://github.com/facebook/mcrouter/wiki/List-of-Route-Handles#missfailoverroute) for more information.
* New feature that saves the last valid configuration to disk and falls back to it when mcrouter starts with an invalid config. Mcrouter will trust this saved config for up to one hour (configurable). More information in this [wiki page](https://github.com/facebook/mcrouter/wiki/Command-line-options).
* Bug fixes and performance improvements.

