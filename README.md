# hass-auto-update

A minimal Home Assistant automation that auto-installs every pending update —
Core, OS, Supervisor, add-ons, and HACS-managed integrations — as soon as
it's available, with an hourly resweep so a missed event or a failed install
attempt gets retried automatically.

18 lines, no per-integration list to maintain, running in production for
months.

## Why

Home Assistant surfaces pending updates as `update.*` entities, but actually
installing them is still a manual click per entity — unless you enable
auto-update on each one individually and remember to do it again every time
a new integration or add-on shows up. This automation covers the whole
`update` domain generically instead.

## The automation

```yaml
- id: auto_install_update_on_available
  alias: Auto Install Updates
  description: >-
    Install any update.* entity as soon as it becomes available, and
    re-sweep every hour as a self-healing backstop so a single missed
    event or failed install attempt gets retried automatically. Scoped to
    the update domain via template (no per-entity/per-integration list to
    maintain as new update.* entities appear).
  triggers:
  - platform: template
    id: update_available
    value_template: >-
      {{ states.update | selectattr('state', 'eq', 'on') | map(attribute='entity_id') | sort | join(',') }}
  - platform: time_pattern
    id: periodic_sweep
    hours: '/1'
  conditions: []
  actions:
  - repeat:
      for_each: >-
        {{ states.update | selectattr('state', 'eq', 'on') | map(attribute='entity_id') | list }}
      sequence:
      - action: update.install
        continue_on_error: true
        target:
          entity_id: '{{ repeat.item }}'
      - delay: '00:00:30'
  mode: queued
  max: 10
```

Also available as a standalone file: [`auto_install_updates.yaml`](auto_install_updates.yaml).

## How it works

- **Template trigger** watches `states.update` for anything in the `on`
  (update available) state — no per-entity or per-integration list to keep
  in sync as your setup grows.
- **Hourly `time_pattern` resweep** is a self-healing backstop: if the
  template trigger's event gets missed, or an install attempt fails, the
  next hourly pass picks it back up.
- **`continue_on_error: true`** means one failing update doesn't block the
  rest of the batch.
- **`mode: queued, max: 10`** means if the automation re-triggers while a
  previous run is still working through its list, the new run queues
  instead of getting dropped or clobbering the run in progress.

## Before you use this

This automation has no backup logic of its own — on purpose. It's designed
to pair with Home Assistant's own **Automatic Backups**
(Settings → System → Backups → enable "Automatic backups"), not duplicate
it. Turn that on first, with retention high enough to give yourself a few
days of rollback margin. Without it, an update that eventually breaks
something has no easy way back.

(Home Assistant Supervisor also appears to create a backup automatically
before updating individual add-ons, independent of this automation — so
add-on updates specifically may already have their own safety net on top of
your scheduled backups.)

## Install

Paste the automation into `automations.yaml` directly, or add it via
**Settings → Automations → Edit in YAML** in the UI, then reload
automations (or restart).

## License

MIT — see [LICENSE](LICENSE).
