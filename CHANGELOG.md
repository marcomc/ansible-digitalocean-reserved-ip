# Changelog

## Unreleased

### Added

- Bootstrapped the new `ansible-digitalocean-reserved-ip` repository from the
  historical reserved-IP work in `ansible-digitalocean-floating-ip`.
- Added Galaxy-ready role metadata and Reserved-IP documentation.
- Added a repository-specific `AGENTS.md` aligned with this standalone Ansible
  role's linting and documentation workflow.
- Added Reserved-IP-only variables and facts, removing the temporary
  Floating-IP compatibility aliases from the copied code.
- Added renamed task files that use Reserved-IP terminology consistently across
  the role internals.
- Added metadata-only anchor facts so the role aligns with default-route-based
  Reserved IP routing without the copied legacy SNAT behavior.
- Added a slimmer task graph by removing the unused iptables helper tasks and
  the metadata wrapper task.
- Added a reorganised README with a table of contents, refreshed publication
  and authorship details, simpler generic consumer examples, and
  Reserved-IP-consistent TODO wording.
- Added a refreshed `tests/README.md` that describes the current Reserved-IP
  test harness instead of the copied Floating-IP and Travis-era workflow.
- Added a slimmer test harness by removing the unused copied firewall-era test
  dependency files.
- Added a TODO item to track future CI setup for the role on the private
  Jenkins instance.
- Added a `tests/test_variables.example.yml` file and a collection-based
  `tests/requirements.yml` to make the live test harness easier to configure.
- Added compact maintainer, support, and repository-status sections to the
  README so publication and ownership details are easier to find.
- Added the exact `doctl compute ssh-key list` commands to the test
  documentation so local `do_ssh_keys` values are easier to discover.
- Added the exact recommended live test command sequence to `tests/README.md`,
  including Debian-first validation, full-matrix execution, and cleanup.
- Added support for `DO_OAUTH_TOKEN` as a test-harness fallback when
  `DIGITAL_OCEAN_API_TOKEN` is not exported, and documented the related
  troubleshooting step for `401 Unauthorized` failures.
- Added support for storing the DigitalOcean test token as `do_test_api_token`
  in the gitignored `tests/test_variables.yml`, and clarified that the current
  live test harness does not require AWS credentials.

### Changed

- Removed the stale Travis-era ignored key files from `.gitignore`.
- Updated the live test inventories to current DigitalOcean images for Debian
  13, Ubuntu 24.04 LTS, and CentOS Stream 10.
- Switched the test helper role to the supported
  `community.digitalocean.digital_ocean_droplet` module and refreshed the
  Python bootstrap tasks for current Debian, Ubuntu, and CentOS-family hosts.
- Clarified in `tests/.gitignore` that `tests/test_variables.yml` is kept
  untracked to prevent accidental commits of local live-test overrides.
