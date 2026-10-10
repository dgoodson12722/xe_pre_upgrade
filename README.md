# IOS XE upgrade (interactive)

Steps 1 to 7, get the image onto one IOS XE device's flash and verify it.
Works on any IOS XE platform (ASR1k, ISR4k, C8000, Catalyst 9000, ...): the file system is detected.
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
2. Username and password, then checks SSH, logs in, shows hostname and version, asks "Is this the right device?"
3. Image file name, e.g. `cat9k_iosxe.17.12.04.SPA.bin`
4. MD5 or SHA512 (type is detected from the length)
5. Runs `dir` to find the default file system (`bootflash:` or `flash:`) and checks it for the image; if present, goes to step 7
6. Asks `me` or `playbook` for the copy. `playbook` then asks `push` (from this server, path asked) or `pull` (device fetches from an SCP server; server, directory, username and password asked). Stops if flash is too small.
7. Runs `verify /md5` or `verify /sha512`. On a mismatch asks `me` or `playbook` to delete the bad file, then stops.

Add `-vvv` when debugging to see each command sent to the device.

## Layout

```
playbooks/01_stage_image.yml   steps 1 to 7
inventory/hosts.yml            localhost only; the device is asked for at run time
tasks/  filter_plugins/  tests/   empty for now; filled as later steps are agreed
images/                        optional place for the image when you push it from this server
reports/                       run-time output (not committed)
```

Run every command from this directory so `ansible.cfg` is picked up.

## Notes for debugging

- Passwords are never printed; tasks that use them have `no_log: true`.
- `host_key_checking` is off in `ansible.cfg` so a first connection to a device does not stop on an unknown key. Turn it on if your policy needs it.
- A pull needs at least 2 GB free (`pull_min_free_bytes` in the playbook), because the size is not known before the copy.
- The device uses its default SSH source for a pull. If it sets `ip ssh source-interface` to a management-VRF port, check the pull reaches the SCP server on the first run.
# pre_xe_upgrade
# xe_pre_upgrade
