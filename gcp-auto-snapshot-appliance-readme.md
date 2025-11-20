# Auto Snapshot Appliance
## Overview
A lightweight VM-based tool that automatically creates and manages persistent disk snapshots across a Google Cloud project. The appliance is designed for simplicity, zero maintenance, and easy deployment.

This appliance automatically snapshots VMs in your Google Cloud project.

- Runs entirely inside your project.
- Uses the VM’s attached service account — no JSON key needed.
- Runs automatically via systemd timer: appliance.timer.
- Configurable via /opt/appliance/config.yaml.

No external services, backend, or SaaS platform is required.

## Key Features
- Automated snapshots at configurable intervals  
- Retention policy cleanup  
- Tag-based filtering (`include_tags`, `exclude_tags`)  
- Runs via `systemd` timer with no cron or extra dependencies  
- Zero-maintenance and zero external calls  
- Fully contained within customer’s GCP environment  

## Architecture
```
+----------------------------+
| GCE VM (Appliance Image)   |
|----------------------------|
| systemd timer → main.py    |
| Python compute API client  |
+----------------------------+
          |
          v
+----------------------------+
| GCP Compute API            |
| - List instances           |
| - Create snapshots         |
| - Delete old snapshots     |
+----------------------------+
```

## Installation
Deploy the appliance by creating a new VM using the Marketplace-provided image.

### Requirements
- A GCP project with Compute Engine enabled  
- Permissions for the VM's service account:
  - `roles/compute.admin`

### Configuration
Edit `/opt/appliance/config.yaml`

**Example config.yaml:**
frequency_hours: 24
retention_days: 7
include_tags:
  - backup
  - auto-snap
exclude_tags:
  - no-backup
  - test
target_vms: []
log_path: "/opt/appliance/logs/appliance.log"
project: "YOURPROJECT"
zone: "us-central1-a"

### Notes:

**Tags:** Used to select which VMs to snapshot.
**Target VMs:** Optional; overrides tags.
**Logs:** All actions are recorded in log_path.

### Running the Appliance
**Automatic:** Runs via systemd timer every frequency_hours.
**Manual test run:**
sudo python3 /opt/appliance/main.py

## How It Works
The `systemd` timer triggers `main.py`, which:
1. Reads configuration  
2. Lists VM instances  
3. Evaluates tag filters  
4. Creates snapshots  
5. Deletes expired snapshots  

Full code lives in `/opt/appliance/`.

**Check logs:**

sudo tail -f /opt/appliance/logs/appliance.log

### Systemd Commands
**Check status**
sudo systemctl status appliance.timer
sudo systemctl status appliance.service

**Force a reload after updating code**
sudo systemctl daemon-reload
sudo systemctl restart appliance.timer

### Notes for Developers
No JSON key is needed; the appliance uses metadata-based credentials.
Use gcloud compute instances add-tags VM_NAME --tags=backup to mark VMs for backup.
Use /opt/appliance/config.yaml to tweak frequency, retention, or target VMs.
Keep /opt/appliance/logs accessible for debugging.

### Cleanup
To remove snapshots manually:
gcloud compute snapshots list --filter="name~'VM_NAME'" --format="value(name)" | \
xargs -n1 gcloud compute snapshots delete --quiet

