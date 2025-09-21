# ansible-role-lego

![GitHub](https://img.shields.io/github/license/cosandr/ansible-role-lego) ![GitHub last commit](https://img.shields.io/github/last-commit/cosandr/ansible-role-lego) ![GitHub issues](https://img.shields.io/github/issues-raw/cosandr/ansible-role-lego)

**Ansible role for setting up [lego](https://go-acme.github.io/lego).**

## Supported Platforms

| OS Family | Distribution  | Latest | Supported Version(s) | Comment |
|-----------|---------------|--------|----------------------|---------|
| RedHat    | RHEL          | :heavy_check_mark: | 9 | |
|           | RockyLinux    | :heavy_check_mark: | 8, 9 | |
|           | AlmaLinux     | :heavy_check_mark: | 8, 9 | |
|           | Fedora        | :heavy_check_mark: | 36, 37, 38 | |

## Requirements

Ansible 2.12 or higher.

## Variables

| Name                                  | Default Value       | Description                                                              |
|---------------------------------------|---------------------|--------------------------------------------------------------------------|
| `lego_version`                        | latest              | Set Lego version, can be specific version, i.e. `v4.14.2`                |
| `lego_renew_oncalendar`               | `*-*-* 00/12:00:00` | Sets systemd timer schedule                                              |
| `lego_renew_random_delay`             | 1h                  | Sets systemd timer random delay                                          |
| `lego_pre_script`                     | ""                  | Raw string inserted in lego-wrapper                                      |
| `lego_home`                           | /etc/lego           | Path to store files, including accounts and certificates                 |
| `lego_user`                           | lego                | User to run Lego as                                                      |
| `lego_group`                          | lego                | Group to run Lego as                                                     |
| `lego_domains`                        | []                  | Domains to get certs for, see example                                    |
| `lego_provisioning_synced`            | true                | Delete renew hooks which are no longer defined                           |
| `lego_renew_hooks`                    | []                  | Hooks to run after renewing, see example                                 |
| `lego_service_sandbox_readwritepaths` | []                  | Paths that won't be restricted by systemd sandboxing of the lego service |
| `lego_service_sandbox_disable`        | false               | Completely disable systemd sandboxing of the lego service                |



## Dependencies

None.

## Example Playbook

```yaml
---

- hosts: all
  become: true
  gather_facts: true
  roles:
    - role: lego
      vars:
        lego_domains:
          - name: "example.com"
            email: you@example.com
            dns: rfc2136
            env:
              RFC2136_NAMESERVER: 127.0.0.1
              RFC2136_TSIG_KEY: lego
              RFC2136_TSIG_ALGORITHM: hmac-sha256.
              RFC2136_TSIG_SECRET: YWJjZGVmZGdoaWprbG1ub3BxcnN0dXZ3eHl6MTIzNDU=
            options:
              global:
                - --dns.resolvers 9.9.9.9:53
              command:
                - --preferred-chain "ISRG Root X2"
        lego_renew_hooks:
          # Hooks prefixed with sudo run as root
          - name: sudo-restart-nginx
            content: |
              #!/bin/sh
              echo "I am root"
              systemctl restart nginx
          # Anything else runs as lego user
          - name: something-else
            content: |
              #!/bin/sh
              echo "I am not root"
```

## Author

[Andrei Costescu](https://github.com/cosandr/)
