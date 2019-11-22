SAP Cloud Platform (SCP) is an open platform-as-a-service (PaaS) product that provides core services, for building and extending cloud applications on multiple cloud IaaSes. SCP supports AWS, Azure, GCP, and Alibaba Cloud.

> One of the core services provided by SCP is PostgreSQL as a Service (PostgreSQL-as-a-Service). Each PostgreSQL-as-a-Service instance consists of 5 VMs - Postgres-Master, Postgres-Standby, and 3-PGPOOL VMs. Data is replicated asynchronously from Postgres-Master to Postgres-Standby.

SCP manages more than 10000 PostgreSQL-as-a-Service instances across multiple IAASs.

## Postgresql-as-a-Service - Backup & Recovery
PostgreSQL-as-a-Service leverages the disk **Snapshots** offering of different IaaSes to take backups. Data directory of the Postgres instance is present on the persistent disk attached to the VMs on which Postgres is running. A snapshot of the persistent disk is created while taking backup. The snapshot service offering by IaaSes comes with in-built support for encryption and replication of Snapshots across multiple facilities. Backups are taken online, which means there is no downtime during the backup. One of the main advantages of physical backup is that it can support **Point-In-Time-Recovery**(PITR). This helps in recovering the database to any timestamp or a particular transaction ID. WAL logs are archived on cloud-provided storage service, to support PITR.

For restore, the backed-up snapshot is used to **create disks**. These disks are **_mounted_** on the PostgreSQL VMs(primary & standby). The old persistent disks are released once the new disks are attached successfully. The service downtime depends on the time taken for creating the disks from the snapshot. On some IaaSes, it depends on the data size, whereas on others it is independent. In case of PITR based restore, WAL logs are replayed after disks are attached.

A service **broker** is used to manage the backup of a large scale of PostgreSQL clusters. It **schedules** the backup of each service instance with some fixed interval(depending on SLAs). Restore is an on-demand event available as self-service to customer. The broker exposes API, to trigger restore on PostgreSQL cluster.

## Centralized Monitoring of Backup & Restore
Backup and Restore related SLAs are agreed upon by customers, hence monitoring of these features are quite important. A **monitoring agent** runs in every PostgreSQL VM to report its health metrics along with **backup status** of all the PostgreSQL cluster.

<img src="https://github.com/akashkumar58/pgconf/blob/master/backup-status.png" width="420" align="left"> <img src="https://github.com/akashkumar58/pgconf/blob/master/backupStatus.png" width="420" float="right">

## Alerting System for Backup & Restore
In case when backup fails or after restore the service instance is not running, email based alerts are raised.
<p align="center">
  <img src="https://github.com/akashkumar58/pgconf/blob/master/backupAlert.png" width="600"/>
</p>
