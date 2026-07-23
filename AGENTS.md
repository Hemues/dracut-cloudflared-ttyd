# Agent Notes

## Documentation Discipline
- Treat documentation as part of the change, not a follow-up. When behavior, commands, deployment flow, debugging steps, requirements, or lessons learned change, update the nearest `README.md` and this `AGENTS.md` in the same change.
- Keep `README.md` focused on user/operator-facing setup, usage, troubleshooting, and release/deployment steps.
- Keep `AGENTS.md` focused on future-agent context: architecture traps, verified commands, gotchas, environment details, and lessons learned.
- If a repo also has `LESSONS-LEARNED.md`, record durable postmortems there too and cross-reference from `README.md`/`AGENTS.md`.
- Before finishing, check that docs reflect what was actually tested, committed, released, or deliberately skipped.

## What This Directory Is
This builds and packages a Fedora dracut module that brings up networking in initramfs, starts a Cloudflare tunnel, and exposes a ttyd web terminal for remote LUKS unlock.

## Start Here
- Read `README.md` for install/configuration and kernel command line requirements.
- Read `LESSONS-LEARNED.md` before editing dracut modules, NetworkManager profile copying, LVM activation, or tunnel startup.
- Packaging is RPM/spec driven; inspect `dist/` and the spec before changing release behavior.

## Known Traps (see LESSONS-LEARNED.md)
- **Interface naming is not guaranteed in the initramfs.** Copied NM profiles are
  bound to `interface-name=`; if predictable naming flakes (NIC comes up as
  `eth0`/`eth1` instead of e.g. `ens6f0`) the profiles silently don't match and
  the tunnel never starts. `module-setup.sh` now pins physical NICs by MAC via a
  generated `.link` (`_pin_copied_iface_names`) — keep it. When diagnosing "tunnel
  didn't come up", check the interface *name* in the `net-detect` journal first
  (LESSONS #8).
- The deployed RPM can lag the repo source: the initramfs `cloudflared.service`
  used `head` (absent from initramfs); repo source uses `sed -n`. Rebuild/reinstall
  the RPM after source changes so the baked unit matches.

## Work Safely
- Initramfs failures can make encrypted systems unbootable remotely. Keep rollback instructions current.
- Never `rm -rf` a work tree that may contain bind mounts; verify nested mounts through `/proc/self/mounts` before cleanup.
- Do not commit Cloudflare tunnel tokens, WiFi profiles, SSH keys, or LUKS secrets.
- Preserve `rd.neednet=1` and documented NetworkManager behavior unless testing proves a new boot path.

## Validation
- Build the RPM with the documented `rpmbuild` flow if packaging changed.
- After install/config changes, regenerate initramfs and inspect included files before rebooting.
- Test the no-local-console recovery path in a controlled environment.
