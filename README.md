# IOS XE upgrade (interactive)

Steps 1 to 7, get the image onto one router's bootflash and verify it.
Built for Linux, Python 3.8.10, ansible-core 2.13.13.

## Setup

```
ansible-galaxy collection install -r requirements.yml -p ./collections
pip install paramiko scp      # network_cli logins and the "push" copy
```

## Run

```
ansible-playbook playbooks/01_stage_image.yml
```

It asks for everything as it goes:

1. Device IP or FQDN
2. Username and password, then checks SSH, logs in, shows hostname and version, asks "Is this the right router?"
3. Image file name, e.g. `c8000aep-universalk9.17.12.08.SPA.bin`
4. MD5 or SHA512 (type is detected from the length)
5. Checks `dir bootflash:` for the image; if present, goes to step 7
6. Asks `me` or `playbook` for the copy. `playbook` then asks `push` (from this server, path asked) or `pull` (router fetches from an SCP server; server, directory, username and password asked). Stops if bootflash is too small.
7. Runs `verify /md5` or `verify /sha512`. On a mismatch asks `me` or `playbook` to delete the bad file, then stops.

Add `-vvv` when debugging to see each command sent to the router.

## Layout

```
playbooks/01_stage_image.yml   steps 1 to 7
inventory/hosts.yml            localhost only; the router is asked for at run time
tasks/  filter_plugins/  tests/   empty for now; filled as later steps are agreed
images/                        optional place for the image when you push it from this server
reports/                       run-time output (not committed)
```

Run every command from this directory so `ansible.cfg` is picked up.

## Notes for debugging

- Passwords are never printed; tasks that use them have `no_log: true`.
- `host_key_checking` is off in `ansible.cfg` so a first connection to a router does not stop on an unknown key. Turn it on if your policy needs it.
- A pull needs at least 2 GB free (`pull_min_free_bytes` in the playbook), because the size is not known before the copy.
- The router uses its default SSH source for a pull. These routers set `ip ssh source-interface GigabitEthernet0` (management VRF); check the pull reaches the SCP server on the first run.
# pre_xe_upgrade
# xe_pre_upgrade
