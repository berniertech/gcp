GCP Auto Snapshot Appliance
Overview

GCP Auto Snapshot Appliance is a lightweight virtual machine that automatically creates and manages Google Compute Engine disk snapshots within a Google Cloud project.

The appliance runs entirely inside the customer’s Google Cloud environment and requires no external services or SaaS backend. It is designed for simplicity, transparency, and low operational overhead.

Key Features

Automatic snapshots of Compute Engine boot disks

Snapshot selection based on network tags

Configurable snapshot retention policy

Runs on a schedule using systemd timers

Uses the VM’s default service account (no keys required)

No external dependencies or control plane

How It Works

The appliance runs on a schedule defined by a systemd timer.

On each run, it discovers Compute Engine instances in the project.

Instances are filtered using configured network tags.

Boot disks of eligible instances are snapshotted.

Old snapshots created by the appliance are deleted based on retention settings.

Scheduling

The execution schedule is controlled by systemd, not by the application code.

By default:

The appliance runs once every 24 hours

First execution occurs shortly after VM boot

To customize the schedule:

sudo systemctl edit appliance.timer
sudo systemctl daemon-reload
sudo systemctl restart appliance.timer

Configuration

Configuration is stored at:

/opt/appliance/config.yaml

Example configuration
# How long snapshots are kept before deletion (in days)
retention_days: 7

# Network tags used to include instances
include_tags:
  - backup
  - auto-snap

# Network tags used to exclude instances
exclude_tags:
  - no-backup
  - test

# Optional: explicitly target specific VM names (overrides tags)
target_vms: []

# Optional: log file location
log_path: "/opt/appliance/logs/appliance.log"

Network Tags vs Labels (Important)

Instance selection is based on Compute Engine network tags, not labels.

Network tags are commonly used for firewall rules.

They are configured under VM → Edit → Networking → Network tags.

Labels are not used by this appliance.

Example CLI check:

gcloud compute instances describe INSTANCE_NAME \
  --zone=ZONE \
  --format="get(tags.items)"

Zones and Regions

Compute Engine virtual machines are deployed in zones.

By default:

The appliance scans all zones in the project to discover instances.

Optionally:

A specific zone can be configured in config.yaml to limit snapshot operations.

This behavior ensures that no instances are missed unless explicitly scoped.

Permissions and Authentication

The appliance uses the default service account attached to the VM.

Required permissions

The service account must have Compute Engine permissions, for example:

roles/compute.admin
(or equivalent fine-grained snapshot permissions)

OAuth scopes

The appliance requires the standard cloud-platform OAuth scope.

When deployed via Google Cloud Marketplace, the required scopes and permissions are automatically configured.

No service account keys are required.

Logs and Troubleshooting

Logs are written to:

/opt/appliance/logs/appliance.log


You can also inspect systemd logs:

journalctl -u appliance.service

Security Model

Runs entirely in the customer’s project

No inbound network access required

No external endpoints or callbacks

No credential material stored on disk

Typical Use Cases

Automated VM backup policies

Small and medium environments without backup tooling

Cost-effective snapshot automation

Compliance-oriented environments requiring local control

Support

This product is provided as a self-contained appliance.

For documentation and updates, refer to the product listing or associated documentation repository.

Versioning

This image represents the initial production release.

Future versions may include:

Label-based selection

Regional scoping

Per-disk policies

Notification integrations
