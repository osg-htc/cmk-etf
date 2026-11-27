# Releasing cmk-etf

The ETF base image: CheckMK Raw with the ETF frameworks (ncgx, nap, nagios-stream) and the `etf` site. [perfsonar-plugins](https://github.com/osg-htc/perfsonar-plugins) builds on it.

## Versions

Releases are numbered `X.Y.Z` and published as `ghcr.io/osg-htc/cmk-etf:X.Y.Z`.

- **Z:** fixes, and rebuilds with updated packages.
- **Y:** a CheckMK patch level, or other changes in behaviour.
- **X:** changes that need action when deploying, such as a CheckMK minor upgrade, a site migration or new required configuration.

A release image is never rebuilt or overwritten. A fix is a new release.

Other image tags:

- `:main` is built only by running the Docker workflow by hand, for testing. Do not deploy it.
- PR builds are not pushed anywhere.
- There is no `:latest`.

## What is pinned

- **CheckMK:** `check_mk_version` in the `Dockerfile`, currently 2.3.0p52. Record it in the release notes.
- **Base image:** `almalinux/9-base:<version>-<date>` in the `Dockerfile`, never `latest`.
- **Not pinned:** the ETF packages (from the CERN etf9 repository) and the OS packages, taken by `yum update`. They are whatever is current at build time, and an image, once released, keeps them.

Change a pin in a PR, then release.

## Cutting a release

1. Merge the changes to `main`. **Checks** and the **Docker** build run on every PR.
2. Create the release, which also creates the tag:
   ```
   gh release create X.Y.Z --repo osg-htc/cmk-etf --target main --generate-notes
   ```
   You can also use the GitHub web page: Releases, then "Draft a new release", with a new tag `X.Y.Z`.
3. The tag starts the Docker workflow, which builds, pushes and signs `ghcr.io/osg-htc/cmk-etf:X.Y.Z`. Wait for it to finish (`gh run watch`).
4. In perfsonar-plugins, set `CMK_ETF_VERSION` in its `Dockerfile` to the new version and release it as well. psetf runs perfsonar-plugins, not this image.

## Deploying

psetf does not run this image directly. A cmk-etf release reaches psetf through a perfsonar-plugins release built on it; see perfsonar-plugins' RELEASING.md.

## Changing the CheckMK version

The `etf` site is inside the image. Only `etc/check_mk` (`/opt/cmk/check_mk` on the host) and the perfdata are mounted. So a new image brings a fresh site of the new version, which reads the existing `etc/check_mk`:

- **Within one CheckMK minor (2.3.0pN):** nothing else is needed.
- **Across minors (e.g. 2.3 to 2.4):** check the CheckMK update notes for changes to `etc/check_mk` before deploying, and make it an X release.
