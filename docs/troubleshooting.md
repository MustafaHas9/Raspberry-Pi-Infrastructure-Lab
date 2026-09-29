# Troubleshooting

This document records issues encountered in the lab, including the root cause, resolution, and validation.

---

## SMB Share Fails to Mount After Reboot

### Symptoms

After rebooting `homelab01`, Jellyfin started successfully but media playback failed.

The SMB media share was not mounted at:

`/mnt/jellyfin-media`

### Investigation

Checked the systemd mount unit:

`systemctl status mnt-jellyfin\\x2dmedia.mount`

The mount failed during boot with:

`mount error(113): could not connect to 192.168.4.35`

The Windows media host at `192.168.4.35` became reachable after the Pi had already attempted the mount.

Running `sudo mount -a` after the Windows host was available mounted the share successfully.

### Root Cause

The Pi attempted to mount the SMB share before the Windows SMB server was ready.

The existing `_netdev,nofail` options allowed the Pi to boot without the share, but the failed mount was not automatically retried once the Windows host became available.

### Resolution

Added `x-systemd.automount` to the CIFS entry in `/etc/fstab`.

Current configuration:

`//192.168.4.35/Jellyfin /mnt/jellyfin-media cifs credentials=/etc/samba/jellyfin-credentials,vers=3.1.1,_netdev,nofail,x-systemd.automount 0 0`

This causes systemd to mount the SMB share on demand when `/mnt/jellyfin-media` is accessed.

### Verification

Rebooted `homelab01` without manually mounting the share or restarting Jellyfin.

Jellyfin successfully accessed the media share and playback worked immediately after reboot.

---

## Troubleshooting Template

### Symptoms

### Investigation

### Root Cause

### Resolution

### Verification
