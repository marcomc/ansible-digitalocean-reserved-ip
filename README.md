# Ansible role: DigitalOcean Reserved IP

Manage a DigitalOcean Reserved IP for an existing droplet and expose the
network metadata needed by callers to align routing with the target host.

This repository starts as a copy of the historical
`ansible-digitalocean-floating-ip` work, but it is now packaged and documented as a
Reserved IP role for future publication on Ansible Galaxy.

## Table of contents

- [Overview](#overview)
- [What it does](#what-it-does)
- [Current behavior notes](#current-behavior-notes)
- [Requirements](#requirements)
- [Installation](#installation)
- [Variables](#variables)
- [Exported facts](#exported-facts)
- [Examples](#examples)
- [Development notes](#development-notes)
- [Repository guidance](#repository-guidance)
- [Maintainer](#maintainer)
- [Support](#support)
- [Repository](#repository)
- [License](#license)

## Overview

The role is intentionally narrow in scope:

- manage a Reserved IP for an existing droplet
- expose the reserved-address and anchor-routing metadata needed by callers
- avoid legacy host-level NAT or route-persistence behavior inside the role

## What it does

- attaches an existing Reserved IP to a droplet, or creates one on the fly
- exposes the assigned address as Ansible facts
- retrieves the droplet anchor address and anchor gateway metadata
- does not install legacy SMTP or general TCP `iptables` SNAT rules

## Current behavior notes

- Preferred terminology in this repo is **Reserved IP**.
- This role uses Reserved-IP-only variable names and exported facts.
- DigitalOcean still exposes the underlying API through `/floating_ips` REST
  endpoints; that API naming is preserved where required by the platform.
- The role assumes outbound traffic should follow the default route via the
  anchor gateway rather than host-level SNAT rules.

## Requirements

- Ansible Core $\ge 2.13$
- a DigitalOcean API token with permission to manage droplets and Reserved IPs

## Installation

### Current repository role names

- Local development role name in this repository:
  `inviqa_digitalocean_reserved_ip`
- Intended Ansible Galaxy FQCN from the current repository metadata:
  `inviqa.digitalocean_reserved_ip`

For generic downstream playbook examples, this README uses the simple role name
`digitalocean_reserved_ip`.

If you install the role under a different local name or through a namespace,
use the role name or FQCN that matches your own installation method.

### Publication status

The repository is being prepared for publication. Until that release exists,
it may still be consumed from a local checkout during migration work.

## Variables

| Variable | Default | Notes |
| --- | --- | --- |
| `digital_ocean_api_base_url` | `https://api.digitalocean.com/v2` | Base DigitalOcean API URL. |
| `digital_ocean_api_token` | `{{ enc_do_v2_api_key }}` | API token used for DigitalOcean API requests. |
| `digital_ocean_droplet_id` | `{{ do.droplet.id \| mandatory }}` | Target droplet identifier. |
| `digital_ocean_reserved_ip` | `""` | Preferred Reserved IP input. Leave empty to allocate one automatically. |
| `digital_ocean_metadata_anchor_ipv4_gateway_url` | `http://169.254.169.254/metadata/v1/interfaces/public/0/anchor_ipv4/gateway` | Metadata endpoint used to discover the anchor gateway. |

## Exported facts

The role exposes these Reserved-IP facts for callers:

- `do_droplet_reserved_ip_address`
- `reserved_ip_is_assigned_to_this_droplet`
- `anchor_ip_associated_to_droplet`
- `anchor_gateway_associated_to_droplet`

## Examples

```yaml
---
- name: Attach an existing Reserved IP
  hosts: digitalocean_droplets
  vars:
    reserved_ip_role_name: digitalocean_reserved_ip
  tasks:
    - name: Manage a Reserved IP with your installed role name
      ansible.builtin.include_role:
        name: "{{ reserved_ip_role_name }}"
      vars:
        digital_ocean_api_token: "{{ lookup('env', 'DO_OAUTH_TOKEN') }}"
        digital_ocean_droplet_id: "{{ digitalocean_droplet.id }}"
        digital_ocean_reserved_ip: "139.59.203.219"

- name: Allocate a Reserved IP on the fly
  hosts: digitalocean_droplets
  vars:
    reserved_ip_role_name: digitalocean_reserved_ip
  tasks:
    - name: Allocate and attach a Reserved IP with your installed role name
      ansible.builtin.include_role:
        name: "{{ reserved_ip_role_name }}"
      vars:
        digital_ocean_api_token: "{{ lookup('env', 'DO_OAUTH_TOKEN') }}"
        digital_ocean_droplet_id: "{{ digitalocean_droplet.id }}"
```

If you install the published role under its current namespace, replace
`digitalocean_reserved_ip` with `inviqa.digitalocean_reserved_ip`.

## Development notes

- `tests/` contains the copied integration harness and is being refreshed for
  the new role name.
- `tests/README.md` contains the current live test workflow, including the
  exact first-run command sequence for Debian-first validation, full-matrix
  execution, and cleanup.
- The repo is being prepared for Galaxy publication, so the metadata and
  documentation are intentionally kept explicit.

## Repository guidance

- `AGENTS.md` defines repository-specific editing, linting, and documentation
  expectations for AI coding agents working in this role.
- `.ansible/` is treated as generated or vendored content and should not be
  hand-edited; update the repository source files instead.

## Maintainer

- Maintainer: Marco Massari Calderone `<marco@marcomc.com>`
- Copyright holder: Inviqa UK Ltd

## Support

- For current maintenance and publication work, contact the maintainer above.
- If and when the role is published publicly, issue-tracking and support paths
  should be documented alongside the published source.

## Repository

- Public repository URL: not configured yet
- Publication status: pending Ansible Galaxy and/or public Git publication

## License

MIT
