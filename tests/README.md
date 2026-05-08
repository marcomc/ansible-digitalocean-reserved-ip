
# Test harness

This directory contains a simple integration-style harness for exercising the
role against real DigitalOcean droplets.

## What it does

The test playbooks:

- provision droplets using the helper role under `tests/roles/digital-ocean`
- install Python so Ansible can manage the created hosts
- run `inviqa_digitalocean_reserved_ip` against those droplets
- tag test resources for easier cleanup
- verify that Reserved IP facts are exposed correctly

## Requirements

Set the required environment variables before running the tests:

```text
DIGITAL_OCEAN_API_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

Additional variables can be supplied through the inventory files under
`tests/`.

## Run the tests

From the `tests/` directory:

```text
ansible-playbook -i inventory playbook.yml
```

You can also target a specific inventory file such as `inventory-centos`,
`inventory-debian`, or `inventory-ubuntu`.

## Clean up test resources

To destroy the test droplets and the Reserved IPs associated with them:

```text
ansible-playbook -i inventory playbook_cleanup.yml
```

## Notes

- The cleanup playbook selects test resources using the `ANSIBLE-TEST` tag.
- This harness is intended for local or CI-driven verification against live
  DigitalOcean resources.
