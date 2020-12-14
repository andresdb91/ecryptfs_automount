# eCryptfs auto-mount PAM script

## Introduction
This script allows to auto-mount custom eCryptfs folders on login and auto-unmount them on last session logout, using a configuration file.

## How it works
It works as a PAM script which must be called on both auth and session.

When called in auth step it will mount configured eCryptfs folders if not already mounted.

In session step it will track open sessions using a file in /tmp and will unmount folders when the last session is exited.

It uses a blacklist to avoid counting systemd-user (since it stays open and doesn't close on logout) and sudo (since it shouldn't need access to encrypted folders and won't have the password to mount them anyways).

## Usage
See sample config and [ArchLinux eCryptfs guide](https://wiki.archlinux.org/index.php/ECryptfs)

1. Copy script to a safe place, give root ownership and remove unnecesary permissions
2. Add script in /etc/pam.d/system-auth to *auth* section after all login checks with *expose_authtok*
   * auth optional pam_exec.so expose_authtok *path_to_script*
3. In the same file, add also to *session* section
   * session optional pam_exec.so *path_to_script*
4. Create a configuration file for each user that will use an encrypted folder
   * Configuration files can be /home/*user*/.ecryptfs/config or /home/.ecryptfs/*user*/config (for full home encryption)
   * Full home encryption *should* get mounted before attempting to read the configuration file inside the home directory so further configurations can be kept encrypted

## Limitation and known issues
* The wrapping passphrase must be the same as the user's password (since it passed through *expose_authtok* in PAM)
* For this same reason it will (probably) only work if the first login is done with a password
* It's only tested on a single home folder and user, it may not work with multiple folders due to missing input from PAM but should work with multiple users (each with only one encrypted folder)

## On systemd-homed
With [systemd-homed](https://wiki.archlinux.org/index.php/Systemd-homed) out, I probably won't fix any of the above issues since it seems to be a better solution in every aspect.

## Disclaimer
Since this is a PAM module it runs early and with very high privileges and it may be susceptible to several attacks that can compromise your system. I do use it but it's not exhaustively tested nor audited, use it at your own risk (and don't use it in an enterprise environment).