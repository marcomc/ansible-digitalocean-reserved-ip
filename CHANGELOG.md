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

### Changed

- Refactored repeated `when` conditions in task files into named blocks to
  reduce verbosity and improve maintainability:
  - Consolidated SSH handoff preparation tasks in
    `tasks/setup_outbound_routing.yml` under "Prepare Reserved IP for SSH
    handoff when metadata available" block.
  - Consolidated Debian-family routing tasks under "Implement Debian-family
    immediate and persistent Reserved IP routing" block.
  - Refactored anchor IP and gateway retrieval logic in
    `tasks/retrieve_anchor_ip.yml` into named "Retrieve and store anchor IP
    address" and "Retrieve and store anchor gateway" blocks.
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
- Added a preflight DigitalOcean `/v2/account` authentication check so invalid
  test tokens fail early with a clearer message before the role tasks run.
- Added outbound routing configuration via the anchor gateway, following the
  official DigitalOcean Reserved IP documentation. Routing is applied
  immediately via `ip route` commands and persisted via netplan (Debian/Ubuntu)
  or NetworkManager plus reboot on CentOS-family hosts.
- Added `enable_reserved_ip_outbound_routing` variable (default `true`) to
  control whether the role configures outbound routing through the Reserved IP.
- Added a pre-cutover SSH handoff so Ansible reconnects through the Reserved IP
  before the default route changes on the droplet.
- Added a netplan handler to support persistent Debian/Ubuntu routing
  configuration.
- Added outbound routing verification to the live test suite, querying the
  droplet's public IP via `curl -4 https://icanhazip.com/` and asserting it
  matches the assigned Reserved IP address.
- Added per-host temporary `known_hosts` handling for the Reserved-IP SSH
  handoff so parallel live-test runs do not race on shared host-key state.
- Added `community.general` as a role/test dependency so RedHat-family
  NetworkManager gateway persistence can use the `community.general.nmcli`
  module.

### Changed

- Removed the stale Travis-era ignored key files from `.gitignore`.
- Updated the live test inventories to current DigitalOcean images for Debian
  13, Ubuntu 24.04 LTS, and CentOS Stream 10.
- Switched the test helper role to the supported
  `community.digitalocean.digital_ocean_droplet` module and refreshed the
  Python bootstrap tasks for current Debian, Ubuntu, and CentOS-family hosts.
- Clarified in `tests/.gitignore` that `tests/test_variables.yml` is kept
  untracked to prevent accidental commits of local live-test overrides.
- Hardened the live test cleanup and validation tasks so unattached Reserved IP
  entries do not crash the test harness when scanning account-wide results.
- Fixed the live test SSH defaults so the harness connects to fresh droplets as
  `root` instead of inheriting the local workstation username.
- Clarified in `tests/README.md` that the private key matching `do_ssh_keys`
  must be loaded into the local SSH agent before running the live tests.
- Fixed the remaining direct `ansible-lint` failures in the test playbooks,
  test task files, and helper-role metadata.
- Clarified in `AGENTS.md` that every created or modified Ansible file,
  including files under `tests/`, must be validated with `ansible-lint`.
- Refined outbound Reserved IP routing so the role now detects the primary
  interface dynamically, applies immediate cutover only on Debian-family
  systems, and persists RedHat-family routing through the active
  NetworkManager connection instead of assuming legacy `ifcfg-eth0` files.
- Replaced command-driven RedHat gateway persistence with
  `community.general.nmcli` and replaced shell-driven netplan handler execution
  with `ansible.builtin.command`.
- Replaced shell-based outbound IP verification in tests with
  `ansible.builtin.uri` while keeping the same Reserved-IP assertion logic.
- Added an explicit `wait_for_connection` stabilization step after
  `reset_connection` so the Reserved-IP SSH handoff is less prone to transient
  reconnect races during parallel matrix runs.
- Consolidated `tests/README.md`: merged the overlapping "One-time setup" and
  "Recommended first run" sections into a single "Setup" section, added a
  table of contents, and removed duplicated steps across sections.
- Changed `tests/group_vars/all/main.yml` so that `DIGITAL_OCEAN_API_TOKEN` /
  `DO_OAUTH_TOKEN` environment variables take precedence over `do_test_api_token`
  in `tests/test_variables.yml`, making the env-var override behaviour explicit.
