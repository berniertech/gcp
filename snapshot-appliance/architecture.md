# Architecture Diagram – GCP Auto Snapshot Appliance

## High-Level Architecture

```
                      +-------------------------------+
                      |      Google Cloud Project     |
                      +-------------------------------+
                                   |
                                   v
+---------------------------------------------------------------+
|                 GCE VM: Auto Snapshot Appliance               |
|---------------------------------------------------------------|
| /opt/appliance/                                               |
|  ├── config.yaml  (frequency, retention, tag filters)         |
|  ├── main.py      (snapshot logic)                            |
|  ├── helpers.py   (disk utils, cleanup logic)                 |
|  └── service files (systemd unit + timer)                     |
|                                                               |
| systemd timer → runs main.py every X hours                    |
+---------------------------------------------------------------+
                                   |
                                   v
+---------------------------------------------------------------+
|                    Google Compute Engine API                  |
|---------------------------------------------------------------|
| - List VM instances                                           |
| - Read instance labels                                        |
| - Create snapshots                                            |
| - Delete expired snapshots                                    |
+---------------------------------------------------------------+
```

## Data Flow Summary
1. `systemd` timer triggers `main.py`.
2. Config is loaded from `/opt/appliance/config.yaml`.
3. The script lists all VM instances in the project.
4. Instances are filtered by `include_tags` and `exclude_tags`.
5. For each valid VM, snapshots are created.
6. Old snapshots are removed based on `retention_days`.
7. Output logs are stored in `journalctl`.

