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
- Added compact maintainer, support, and repository-status sections to the
  README so publication and ownership details are easier to find.

### Changed

- Removed the stale Travis-era ignored key files from `.gitignore`.
