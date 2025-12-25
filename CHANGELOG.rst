Changelog
#########
Unreleased
==========

[1.2.0] - 2025-12-25
====================
Added
-----
* New `-L` / `--linkdest-list` command-line option.
* New `linkdest_list` configuration option.
* Validate that the `working_dir` path exists.
* Validate that all paths in `linkdest_list` exist.

Changed
-------
* Improved validation of `rate_limit` values.

[1.1.0] - 2025-12-06
====================
Added
-----
* Automatic replacement configuration section `[sisyphus-mirror]` to `[sisyphus_mirror]`
  for configuration loading.

Changed
-------
* Default configuration file path changed to `/etc/sisyphus-mirror/sisyphus-mirror.toml`
  instead of `/etc/sisyphus-mirror/default.toml`.

[1.0.0] - 2025-12-04
====================
Added
-----
* First public release.
