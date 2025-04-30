# Comparing Percona software for MongoDB solutions

Percona Software for MongoDB offers a variety of tools for different purposes. Below is a comparison of Percona MongoLink with other Percona tools:

## Primary purpose

| Tool | Primary Purpose | Best Use Cases |
| --- | --- | --- |
| Percona MongoLink | Real-time replication and zero-downtime migration | - Live migration between MongoDB deployments<br>- Zero-downtime upgrades<br>- Disaster recovery |
| Percona Backup for MongoDB (PBM) | Backup and restore solution with guaranteed data consistency | - Backup strategy implementation <br>- Point-in-time recovery<br>- Disaster recovery<br>- Upgrades over several major versions with logical backups<br> |
| File-copy-based initial sync in Percona Server for MongoDB Pro| Replica set data synchronization | - Scaling Percona Server for MongoDB deployments<br>- Make a faster initial sync for new replica set members |

## Key features

| Feature | Percona MongoLink | Percona Backup for MongoDB | File-copy-based initial sync |
| --- | --- | --- | --- |
| Real-time replication | ✅ | ❌ | ❌ |
| Zero-downtime migration | ✅ | ❌ | ❌ |
| Logical data copying | ✅ | ✅ | ❌ |
| Physical data copying | ❌ | ✅ | ✅ |
| Point-in-time recovery | ❌ | ✅ | ❌ |
| Cross-version compatibility | ✅ | ✅ | ❌  |
| Deployment scaling | ❌ | ❌ | ✅ |
