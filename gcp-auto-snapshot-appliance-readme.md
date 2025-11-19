# Auto Snapshot Appliance
## Overview
This appliance automatically snapshots VMs in your Google Cloud project.

Runs entirely inside your project.
Uses the VM’s attached service account — no JSON key needed.
Runs automatically via systemd timer: appliance.timer.
Configurable via /opt/appliance/config.yaml.

## Configuration

Edit the configuration file:
sudo nano /opt/appliance/config.yaml


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

**Check logs:**

sudo tail -f /opt/appliance/logs/appliance.log

### Systemd Commands
# Check status
sudo systemctl status appliance.timer
sudo systemctl status appliance.service

# Force a reload after updating code
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

