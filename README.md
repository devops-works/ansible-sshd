Ansible sshd role
=================

Check defaults/main.yml

Hardening
---------

Defaults aimed at keeping a server reachable when a client misbehaves (a service
account opening connections in a loop, or leaving sessions open).

| Variable | Default | Effect |
| --- | --- | --- |
| `ssh_login_grace_time` | `30` | Seconds allowed to authenticate (OpenSSH default: 120). A slow or stalled login frees its slot sooner. |
| `ssh_max_startups` | `"10:30:100"` | Unauthenticated connections, as `start:drop%:full`. Above 10 pending, sshd drops 30% of new connections at random, rising to 100% at 100. |
| `ssh_per_source_max_startups` | `5` | Same limit per source address, so one client cannot take every slot and lock admins out. Set it below the first number of `ssh_max_startups`. |
| `ssh_sftp_client_alive_interval` | `30` | In the `ssh_sftp_chroot` Match block: how often sshd probes a silent client. |
| `ssh_sftp_client_alive_count_max` | `4` | Number of unanswered probes before the session is closed (here 30 x 4 = 2 min). Transfers keep the session alive; only idle ones are closed. |

`PerSourceMaxStartups` requires **OpenSSH >= 8.5** (Debian 12). The role reads the
installed version with `package_facts` and leaves the directive out on older
releases (Debian 11 ships 8.4), where sshd would refuse to start.

The two ClientAlive values can be overridden per host inside `ssh_sftp_chroot`,
next to `command_options` and `password_authentication`:

```yaml
ssh_sftp_chroot:
  group: sftpusers
  gid: 4242
  client_alive_interval: 60
  client_alive_count_max: 3
```

The sshd config is written with `validate: sshd -t -f %s`: an invalid file is
never installed.

Caveat: `ssh_custom`
-------------------

`ssh_custom` entries are written at the **end** of `sshd_config`, after the
`Match` blocks. They therefore apply only to the last `Match` block, not
globally, and sshd refuses to start if a directive is not allowed inside a
`Match` (`LoginGraceTime`, `MaxStartups`, `AllowUsers`, ...). Use a dedicated
variable for anything global.
