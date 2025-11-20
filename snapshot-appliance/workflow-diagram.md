# Workflow Diagram – Snapshot Automation

```
+-------------+
| systemd     |
| Timer fires |
| (every X hr)|
+------+------+
       |
       v
+------+------+
|  main.py    |
| initializes |
+------+------+
       |
       v
+-------------+
| Load config |
|  YAML       |
+------+------+
       |
       v
+-----------------------+
| List VM instances     |
| via Compute API       |
+-----------+-----------+
            |
            v
+---------------------------+
| Filter instances by tags  |
| include/exclude           |
+-----------+---------------+
            |
            v
+---------------------------+
| Create snapshot per disk  |
+-----------+---------------+
            |
            v
+---------------------------+
| Delete expired snapshots  |
+-----------+---------------+
            |
            v
+---------------------------+
| Write logs to journalctl  |
+---------------------------+
```

