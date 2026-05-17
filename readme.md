# Tailscale on SteamOS

This process is derived from the
[official guide](https://tailscale.com/blog/steam-deck/), but has been tweaked
to make the process smoother and produce an installation that comes up
automatically on boot (no need to enter desktop mode).

## Features

- Places files ["properly"](https://specifications.freedesktop.org/fhs/) in
  /usr/bin and /usr/sbin without disabling the SteamOS readonly seal
- Survives system updates

## What's up with this and tailscale-dev/deck-tailscale?

I started this project based on the blog post linked above. It got adopted into
the Tailscale community organization, then my write access got removed and I
couldn't make updates. And the install method changed to something I consider
messier. So I forked it back and unlinked it.

This is the repo and install method I use on my own Deck.

## Installing Tailscale

1. Clone this repo to your SteamOS machine.
2. Run `sudo ./install.sh` to install Tailscale (or update the existing
   installation).
3. Run `sudo tailscale up --qr --operator=$(whoami) --ssh` to have Tailscale
   generate a login QR code. Scan the code with your phone and authenticate with
   Tailscale to bring your SteamOS machine onto your network.

## Updating Tailscale

1. Git fetch and pull to make sure you're up to date.
2. Run `sudo ./install.sh` again.

This process overwrites the existing binaries, service file, and systemd
extension override, so it's not recommended to tweak those files directly.

You can change settings in:

- systemd: any new override file in `/etc/systemd/system/tailscaled.service.d/`

- launch flags: `/etc/default/tailscaled`

The configuration file at `/etc/default/tailscaled` is left alone, so feel free
to edit those. If something goes wrong, copy those files somewhere else and
re-run the install script to get back to a working state.

## Changing the root filesystem after installing Tailscale

This method for installing Tailscale uses
[`systemd` system extensions](https://man.archlinux.org/man/systemd-sysext.8.en)
to install files in the otherwise read-only SteamOS filesystem. A side-effect is
that the `/usr` and `/opt` directories (and directories like `/bin`, `/lib`,
`/lib64`, `/mnt`, and `/sbin`, that typically link to `/usr` due to
[`/usr` merge](https://www.freedesktop.org/wiki/Software/systemd/TheCaseForTheUsrMerge/)
which SteamOS implements) are read-only while system extensions are active,
_even after running `steamos-readonly disable`_.

If you need to modify files in these directories after installing Tailscale, run
the following commands:

```bash
$ systemd-sysext unmerge
$ steamos-readonly disable
[ make your changes to the rootfs now ]
$ steamos-readonly enable
$ systemd-sysext merge
```

## Common issues

### Broken config file

Symptom: `invalid value "" for flag -port: can't be the empty string`

Resolution: Delete `/etc/default/tailscaled` and re-run installer script.
