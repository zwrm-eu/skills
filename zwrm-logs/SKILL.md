---
name: zwrm-logs
description: |
  View logs from ZWRM microVMs and get shells or files in and out of them. Use this skill when the user wants to see VM console output, tail or stream logs, debug a running machine, SSH into a VM, run a one-off remote command, or copy files to/from a VM. Triggers on "logs", "tail the logs", "stream logs", "console output", "check the error", "ssh into", "shell on the VM", "copy file to the VM", or "debug the machine".
allowed-tools:
  - Bash(zwrm *)
---

# zwrm logs / ssh / sftp

Inspect running VMs: console logs (historical or streamed), interactive SSH, one-off remote commands, and file transfer.

## Logs

```bash
zwrm logs                        # current app (reads zwrm.toml)
zwrm logs --app my-api           # specific app (interleaves all machines)
zwrm logs --machine <id>         # one machine
zwrm logs -f                     # stream in real time (Ctrl+C to stop)
zwrm logs -n 500                 # last 500 lines
```

## SSH

```bash
zwrm ssh                         # interactive shell on the current app's VM
zwrm ssh --app my-api            # pick app explicitly
zwrm ssh --machine <id>          # pick machine explicitly
zwrm ssh -- systemctl status     # run one command and exit
zwrm ssh -t -- top               # force a PTY for full-screen commands
```

## File transfer

```bash
zwrm sftp put ./local.txt /root/remote.txt
zwrm sftp get /var/log/app.log ./app.log
zwrm sftp put -r ./dist /srv/app          # recursive
zwrm sftp ls /srv
```

## Command reference

```text
zwrm logs — View VM console logs
      --app string       Application name
      --app-id string    Application ID
  -f, --follow           Follow log output (stream in real-time)
  -n, --lines int        Number of lines to show (default 100)
      --machine string   Machine ID

zwrm ssh [machine-id] [flags] [-- command [args...]] — SSH into a running VM or run a remote command on it
      --app string       Application name
      --app-id string    Application ID
  -C, --command string   Remote command to run (alternative to "-- command")
      --direct           Connect directly to VM (only works from host)
      --machine string   Machine ID
  -T, --no-tty           Disable PTY allocation
  -t, --tty              Force PTY allocation (e.g. for -- top)
      --user string      SSH user (overrides app config)

zwrm sftp — Transfer files to/from a running VM via SFTP
      --app string       Application name
      --app-id string    Application ID
      --machine string   Machine ID
      --user string      SSH user (overrides app config)

zwrm sftp get REMOTE LOCAL — Download a file or directory from the VM
  -r, --recursive   Recursively copy directories

zwrm sftp ls [REMOTE] — List a remote directory on the VM

zwrm sftp put LOCAL REMOTE — Upload a file or directory to the VM
  -r, --recursive   Recursively copy directories
```

## Tips

- **Short machine IDs work** — no need for the full UUID.
- App-level logs interleave output from all machines; use `--machine` when isolating one instance.
- To grep or time-filter, pipe: `zwrm logs -n 1000 | grep -i error`.
- SSH sessions go through the platform's SSH proxy with certificate auth — no key setup needed on the VM.

## See also

- [zwrm-status](../zwrm-status/SKILL.md) — machine status alongside logs
- [zwrm-deploy](../zwrm-deploy/SKILL.md) — redeploy if you spot issues
