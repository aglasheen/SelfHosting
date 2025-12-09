# SSH

Some notes for how to harden SSH on a server

Inspired by [ Setting up a production ready VPS is a lot easier than I thought. ](https://www.youtube.com/watch?v=F-9KWQByeU0)

## Table of contents
- [Overview](#overview)
- [Create a user](#create-a-user)
- [SSH key setup](#ssh-key-setup-client)
- [SSH server configuration](#ssh-server-configuration-secure-defaults)
- [Post-change checks](#post-change-checks)
- [Tips & further reading](#tips) (see also [References](#references))

## Overview
SSH (secure shell) is used to remotely login to machines and provides a command line interface for command execution. This file serves as a starting point for the first login to creating a new user, hardening ssh and using public-private key login

## Prerequisites
- OpenSSH Client - installed on client machine
- OpenSSH Server - installed on the remote machine

## Create a user
Create a new account for SSH access:
```bash
sudo adduser username
sudo id username           # verify the user exists
sudo passwd username       # change password (if needed)
sudo usermod -aG sudo username
```

## SSH key setup (client)
Generate an SSH keypair on your local machine (if you don't have one):
```bash
ssh-keygen
# private: ~/.ssh/id_rsa (keep secret)
# public:  ~/.ssh/id_rsa.pub  (shareable)
```

Copy the public key to the server:
```bash
ssh-copy-id username@server.example.com
```

## SSH server configuration (secure defaults)
Create a backup of the SSH daemon config before editing:
```bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
```

Recommended settings (edit `/etc/ssh/sshd_config` or a file under `/etc/ssh/sshd_config.d/`):
```text
# /etc/ssh/sshd_config (examples)
PermitRootLogin no
PasswordAuthentication no
PublicKeyAuthentication yes
# Optionally change port:
# Port xxxx
```

Sometimes a VPS provider will have more configuration under `/etc/ssh/sshd_config.d/50-cloud-init.conf`, ensure the same options are set there (or removed) so they don't override your main file.

After changes, restart and check status:
```bash
sudo systemctl restart sshd
sudo systemctl status sshd
```

## Post-change checks
- Keep an active session open until you've confirmed a new connection works.
- Test from another client: `ssh username@server.example.com`
- If locked out, restore the backup: `sudo cp /etc/ssh/sshd_config.bak /etc/ssh/sshd_config && sudo systemctl restart sshd`
