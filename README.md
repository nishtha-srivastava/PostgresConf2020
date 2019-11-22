SAP Cloud Platform (SCP) is an open platform-as-a-service (PaaS) which facilitates creating new or extending existing cloud applications and run them in a secure cloud environment managed by SAP. SCP supports multiple Infrastructure-as-a-Service (IaaS) like AWS, Azure, GCP, Alibaba Cloud and OpenStack.

> One of the core platform backing services provided by SCP is PostgreSQL-as-a-Service. SCP manages 10000+ PostgreSQL-as-a-Service instances across multiple IaaSes. Each PostgreSQL-as-a-Service instance consists of VMs to spawn Postgres clusters consisting of primary and standby nodes and data is replicated asynchronously. BOSH is used to **automate large-scale-postgres-deployments** across different IaaSes.

### Backup & Recovery and Improving RTO

Resumption of normal operations and service levels after unexpected failure or disaster is extremely important for **business continuity**.
Determining and planning for **Recovery Time Objective (RTO)** is critical for business impact analysis.

On SCP, PostgreSQL-as-a-Service employs different methods on different IaaSes to take periodic backups of the DB instances. Data-directory of a Postgres instance is stored on the persistent disk attached to the VMs on which Postgres is running. On many IaaSes, for taking physical filesystem level backup, disk-snapshot offering is leveraged. One of the main advantages of physical backup is that it supports Point-In-Time-Recovery (PITR). The disk-snapshot offering by IaaSes comes with built-in-support for encryption and replication of snapshots across multiple availability-zones and regions. PostgreSQL-as-a-Service has a **backup scheduler** to manage backups at large scale. The scheduler schedules and triggers backups of the Postgres instances in batches at regular intervals depending upon SLAs. Backups are taken online, which means there is no downtime during backup. WAL-logs are archived continuously on cloud-provided storage service to support PITR.

In case of an unexpected failure or unfortunate event, restore is carried out. Restore is available as an on-demand event as self-service for users. Earlier restore process involved stopping of the Postgres instance to be restored, removing the data directory and copying of the data-directory from backup disk to the persistence disk of the Postgres instance. And then replaying WAL-logs if point-in-time-recovery is attempted. This whole process took huge amount of time and **deterministic RTO could not be guaranteed**.
To overcome the copy time of data-directory during restore process which could be really huge depending upon the data directory size.. the way restore was done had to be changed. New restore approach involves creating disk from the backed-up-snapshot of the concerned Postgres instance. This disk is *mounted* on the PostgreSQL VMs (primary & standby). The old disk is discarded once the new disk is attached successfully. In case of point-in-time-recovery, WAL-logs are replayed after disk is attached. This entire restore process has been **automated**. This helped in improving RTO of PostgreSQL-as-a-Service offering by a factor of 25 on SCP on many IaaSes like Azure, GCP.

### Centralized Monitoring of Backup & Restore

A **monitoring-agent** runs on the instances to report health metrics along with **backup statuses**.

![N|Solid](https://github.com/nishtha-srivastava/PostgresConf2020/blob/master/bk.PNG?raw=true)

### Alerting System for Backup & Restore
In case when backup fails or after restore if the service instance is not running, **alerts** are raised.

![N|Solid](https://github.com/nishtha-srivastava/PostgresConf2020/blob/master/alert.png?raw=true)
