
# Test harness

This directory contains the live DigitalOcean integration tests for the role.

## Coverage

The default inventory currently exercises:

| Family | Image slug |
| --- | --- |
| Debian | `debian-13-x64` |
| CentOS family | `centos-stream-10-x64` |
| Ubuntu | `ubuntu-24-04-x64` |

The single-family inventories keep the same current image slugs:

- `inventory-debian`
- `inventory-centos`
- `inventory-ubuntu`

## One-time setup

From this directory, install the required Ansible collection:

```text
ansible-galaxy collection install -r requirements.yml
```

Provide a DigitalOcean API token.

The harness accepts either `DIGITAL_OCEAN_API_TOKEN` or `DO_OAUTH_TOKEN`.
Use whichever one your local environment already manages:

```text
export DIGITAL_OCEAN_API_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

or:

```text
export DO_OAUTH_TOKEN=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
```

You can also store the token in `tests/test_variables.yml` as
`do_test_api_token`. That file is gitignored, so local developer overrides do
not get committed:

```yaml
do_test_api_token: "dop_v1_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
do_ssh_keys:
  - "12345678"
```

Copy `test_variables.example.yml` to `test_variables.yml` and set at least one
DigitalOcean SSH key fingerprint or numeric key ID in `do_ssh_keys`.

The harness connects to newly created droplets as `root`, so `do_ssh_keys`
must reference a DigitalOcean SSH key that is available for root login on the
test droplets.

Before running the playbooks, load the corresponding private key into your
local SSH agent so Ansible can authenticate to the droplets:

```text
ssh-add ~/.ssh/<your-private-key>
```

If you use a different agent workflow, make sure the private key matching the
selected DigitalOcean SSH key is available to `ssh` before starting the tests.

To list the available SSH keys with `doctl`:

```text
doctl compute ssh-key list --format ID,Name,FingerPrint
```

To print the list of avaialble SSH Keys with `doctl`:

```text
doctl compute ssh-key list
```

## Recommended first run

From the repository root:

```text
ansible-galaxy collection install -r tests/requirements.yml
export DIGITAL_OCEAN_API_TOKEN='YOUR_DIGITALOCEAN_TOKEN_HERE'
doctl compute ssh-key list --format ID,Name,FingerPrint
test -f tests/test_variables.yml || cp tests/test_variables.example.yml tests/test_variables.yml
```

If your shell already uses `DO_OAUTH_TOKEN`, you can export that instead.

You can also skip exporting entirely by setting `do_test_api_token` in
`tests/test_variables.yml`.

After updating `tests/test_variables.yml` with a real `do_ssh_keys` value, run
Debian first:

```text
ansible-playbook -i tests/inventory-debian tests/playbook.yml
```

If that passes, run the full matrix:

```text
ansible-playbook -i tests/inventory tests/playbook.yml
```

Then clean up:

```text
ansible-playbook -i tests/inventory tests/playbook_cleanup.yml
```

If the Debian-only run fails, clean up before retrying:

```text
ansible-playbook -i tests/inventory-debian tests/playbook_cleanup.yml
```

## Run the tests

Run the full matrix:

```text
ansible-playbook -i inventory playbook.yml
```

Run one family only:

```text
ansible-playbook -i inventory-debian playbook.yml
ansible-playbook -i inventory-centos playbook.yml
ansible-playbook -i inventory-ubuntu playbook.yml
```

## Clean up

Destroy the test droplets and their Reserved IPs:

```text
ansible-playbook -i inventory playbook_cleanup.yml
```

## Outbound routing verification

The test playbooks verify that outbound routing through the Reserved IP is
working correctly by querying the droplet's outbound public IP using
`curl -4 https://icanhazip.com/` and asserting that the returned IP matches
the assigned Reserved IP address.

During the SSH handoff, the role switches Ansible management to the Reserved IP
before the route cutover and uses a per-host temporary `known_hosts` file on
the control machine so parallel matrix runs do not race on shared host-key
state.

On CentOS-family test droplets, the role may reboot once after updating the
active NetworkManager connection with `nmcli connection modify ...
ipv4.gateway ...` so the persisted gateway change is applied cleanly.

If routing verification fails:

1. Check that the droplet has network connectivity
2. Verify DNS resolution is working on the droplet
3. Confirm that the anchor gateway metadata was retrieved correctly
4. Check the droplet's routing table: `ip route show`
5. On the droplet, manually test the reserved IP endpoint:
   `curl -4 https://icanhazip.com/`

To skip routing configuration (though the role enables it by default), add
`enable_reserved_ip_outbound_routing: false` to the playbook variables.

## Notes

- The harness provisions real droplets and Reserved IPs, so it incurs cost.
- The current harness only uses DigitalOcean credentials; no AWS credentials
  are required.
- The harness connects to test droplets as `root`.
- Test droplets are tagged with `ANSIBLE-TEST` for cleanup.
- If a test run fails mid-flight, run the cleanup playbook before starting the
  next run.
- If you get `Permission denied (publickey)` during droplet bootstrap, confirm
  that the private key matching `do_ssh_keys` is loaded in your SSH agent.
- If you get a `401 Unauthorized` error from the DigitalOcean API, verify that
  `do_test_api_token` is set in `tests/test_variables.yml`, or that
  `DIGITAL_OCEAN_API_TOKEN` / `DO_OAUTH_TOKEN` is exported in the same shell
  where you run `ansible-playbook`.
- The harness now checks `/v2/account` before provisioning or cleanup, so an
  invalid token should fail early with a clearer authentication message.
