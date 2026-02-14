# PixEagle ARK-OS Integration — PR Reference

> **Status:** PR #53 was closed (self-closed for revisions). Will resubmit as new PR.
> **Original PR:** https://github.com/ARK-Electronics/ARK-OS/pull/53
> **Created:** 2026-02-14T07:11:43Z
> **Closed:** 2026-02-14T13:32:56Z
> **Base:** main
> **Original branch:** feat/pixeagle-integration (deleted from remote)
> **Backup branch:** backup/pixeagle-integration (this branch)

---

## Original PR Title

```
feat: add PixEagle as managed service
```

## Original PR Body

### Summary
- Register PixEagle (AI vision tracking) as an ARK-OS managed service
- Add nginx proxy for dashboard (`/pixeagle/`) and API (`/pixeagle-api/`)
- Add sidebar link in ARK-UI for quick access
- Add MAVLink routing endpoint for MAVSDK (UDP 14540)

### Architecture

```
PixEagle (~/PixEagle)
├── Backend API        :5077  ──> proxied at /pixeagle-api/
├── React Dashboard    :3040  ──> proxied at /pixeagle/
├── MAVLink2REST       :8088  ──> receives from mavlink-router :14569
├── MAVSDK Server             ──> receives from mavlink-router :14540
└── WebSocket          :5551
```

### Changes (9 files, +117 lines)

| File | Change |
|------|--------|
| `services/pixeagle/pixeagle.manifest.json` | New — service metadata |
| `services/pixeagle/pixeagle.service` | New — systemd user unit |
| `services/pixeagle/install_pixeagle.sh` | New — clone + non-interactive init |
| `default.env` | Add `INSTALL_PIXEAGLE="n"` |
| `tools/install_software.sh` | Add interactive prompt + summary |
| `services/mavlink-router/main.conf` | Add `pixeagle_mavsdk` UDP endpoint |
| `frontend/ark-ui.nginx` | Add `/pixeagle/` + `/pixeagle-api/` proxy |
| `frontend/ark-ui/ark-ui/src/App.vue` | Add sidebar link |
| `README.md` | Add PixEagle to services list |

### Test Plan
- [x] Service installs via `service_control.sh install pixeagle`
- [x] Service appears in ARK-OS web UI with start/stop/autostart controls
- [x] Dashboard loads at `http://jetson.local/pixeagle/`
- [x] API accessible at `http://jetson.local/pixeagle-api/`
- [x] journalctl shows startup banner with version/URLs
- [x] Standalone PixEagle unaffected (no env vars = no behavior change)
- [x] Existing ARK-OS services unaffected
- [x] Sidebar shows both Flight Review and PixEagle links grouped at bottom

---

## Commits (6)

1. **c5f2815** `feat: add PixEagle as managed service`
   - Core integration: manifest, service unit, install script, default.env, install_software.sh, mavlink-router, nginx

2. **c7cd19e** `fix(pixeagle): migrate standalone service to ARK-OS managed service`
   - Detect/disable pre-existing system-level pixeagle.service to prevent conflict

3. **78c53cf** `docs: add PixEagle to services list in README`

4. **6147924** `style(pixeagle): simplify service unit to match ARK-OS conventions`
   - Remove non-standard directives (WorkingDirectory, ExecStop, KillMode, Timeout*, Environment)
   - Match minimal pattern of flight-review/rtsp-server

5. **b9123ae** `fix(pixeagle): use PUBLIC_URL for proper subpath asset loading`
   - Replace nginx sub_filter with CRA PUBLIC_URL mechanism
   - Install script creates `.env.production.local` before init

6. **ae65726** `feat(pixeagle): add dashboard link to sidebar`
   - External link with fa-crosshairs icon, grouped with Flight Review

---

## Key Design Decisions

1. **External clone (not submodule)** — PixEagle is large, independently versioned, has heavy init
2. **configFile: ""** — PixEagle has its own dashboard for config (YAML too complex for TOML wrapper)
3. **PUBLIC_URL=/pixeagle** — Set via `.env.production.local` before dashboard build for proper subpath serving
4. **Minimal service unit** — Matches ARK-OS conventions exactly (flight-review/rtsp-server pattern)
5. **INSTALL_PIXEAGLE default: "n"** — Heavy optional service, user must opt-in
6. **Standalone service migration** — Detects/disables system-level service if present

## Known Issues / TODOs for Resubmission

- [ ] Review if any changes needed based on upstream ARK-OS updates
- [ ] Consider squashing into fewer commits for cleaner history
- [ ] Version tracking / update UI — deferred to separate platform-level PR
- [ ] Verify on fresh Jetson/Pi install (tested on existing setup)

## PixEagle-Side Changes (separate repo)

These changes were committed to `c:\Users\Alireza\PixEagle` main branch:
- `scripts/init.sh` — Added PIXEAGLE_INSTALL_PROFILE + PIXEAGLE_NONINTERACTIVE support
- `scripts/service/run.sh` — Added `log_startup_info()` for journalctl startup banner
