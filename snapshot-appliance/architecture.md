# Architecture Diagram – GCP Auto Snapshot Appliance

                        ┌────────────────────────────────────────────┐
                        │              Google Cloud                  │
                        └────────────────────────────────────────────┘

                                (Customer Tenant – Customer Project)
                                ****************************************
                                *   All runtime components execute     *
                                *   inside the customer’s GCP project *
                                ****************************************
                                                │
                                                ▼
                        ┌────────────────────────────────────────────┐
                        │ Customer GCP Project (Customer Tenant)     │
                        │────────────────────────────────────────────│
                        │ GCE VM: Auto Snapshot Appliance            │
                        │ • Runs Debian-based VM from Marketplace    │
                        │ • /opt/appliance/* code executes here      │
                        │ • systemd timer triggers main.py           │
                        │ • Uses instance-attached service account   │
                        │   (metadata-based auth, no keys)           │
                        └────────────────────────────────────────────┘
                                                │
                                                ▼
                        ┌────────────────────────────────────────────┐
                        │ Google Compute Engine API (Customer Tenant)│
                        │────────────────────────────────────────────│
                        │ • List VM instances in customer project    │
                        │ • Read tags/labels for filtering           │
                        │ • Create disk snapshots                    │
                        │ • Delete expired snapshots                 │
                        └────────────────────────────────────────────┘

                                (No Partner Tenant Runtime Components)
                                ───────────────────────────────────────
                                • Partner provides VM image only
                                • No partner database, backend, SaaS
                                • No external data flows
                                • No customer data leaves GCP

