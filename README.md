# Improving RTO at Enterprise scale for PostgreSQL-as-a-Service on SAP Multi Cloud Platform

SAP Cloud Platform (SCP) is an open platform-as-a-service (PaaS) which facilitates creating new cloud applications or extending existing applications and run them in a secure cloud environment managed by SAP. The SAP Cloud Platform integrates data and business processes. SCP supports multiple Infrastructure-as-a-Service (IaaS) like AWS, Azure, GCP, Alibaba Cloud and OpenStack.

> One of the core platform backing services provided by SCP is PostgreSQL-as-a-Service. SCP manages more than 10000 PostgreSQL-as-a-Service instances across multiple IaaSs. Each PostgreSQL-as-a-Service instance consists of VMs to have Postgres cluster - Postgres-Master, Postgres-Standby. Data is replicated asynchronously from Postgres-Master to Postgres-Standby. BOSH is used to automate large-scale Postgres deployments across different IaaSs.

## Postgresql-as-a-Service - Backup & Recovery and Improving RTO
Resumption of normal operations and service levels after unexpected failure or disaster is extremely important for **business continuity**.
Determining and planning for **Recovery Time Objective (RTO)** is critical for business impact analysis.

PostgreSQL-as-a-Service employs different methods on different IaaSs to take periodic backups of the DB instances. Data directory of a Postgres instance is stored on the persistent disk attached to the VMs on which Postgres is running. On many IaaSs, for taking physical filesystem level backup, disk snapshot offering is leveraged. One of the main advantages of physical backup is that it can support Point-In-Time-Recovery (PITR). The disk snapshot offering by IaaSs comes with built-in support for encryption and replication of snapshots across multiple facilities, zones and regions. PostgreSQL-as-a-Service has a **backup scheduler** to manage backups at a large scale. The scheduler schedules and triggers backups of the Postgres instances in batches at regular intervals dependenting upon SLAs. Backups are taken online, which means there is no downtime during the backup. WAL logs are archived continuously on cloud-provided storage service, to support PITR.

In case of an unexpected failure or unfortunate event, restore is carried out. Earlier restore process involved creating disk out of the snapshot, stopping of the Postgres DB instance to be restored, removing the data directory and copying of the data directory from the backup disk to the  persistence disk of the Postgres instance. And then replaying WAL logs if Point in time recovery is attempted.
This whole process took huge amount of time and **deteministic RTO could not be guarenteed**.
To overcome the data directory copy time during restore process which could be really huge depending upon the data directory size.. the way restore was done was changed. Now restore involves creating disk from the backed-up snapshot of the concerned Postgres instance. This disk is *mounted* on the PostgreSQL VMs (primary & standby). The old persistent disk is released once the new disk is attached successfully. This whole restore process has been automated. The DB service downtime depends on the time taken for creating the disks from the snapshot. In case of PITR based restore, WAL logs are replayed after disks are attached. This helped in improving RTO of PostgreSQL-as-a-Service offering by a factor of 25 on SCP on many IaaSs like Azure, GCP. Restore is an on-demand event available as self-service to customer. 

## Centralized Monitoring of Backup & Restore
Backup and Restore related SLAs are agreed upon by customers, hence monitoring of these features are quite important. A **monitoring agent** runs in every PostgreSQL VM to report its health metrics along with **backup status** of all the PostgreSQL cluster.

<img src="https://github.com/akashkumar58/pgconf/blob/master/backup-status.png" width="420" align="left"> <img src="https://github.com/akashkumar58/pgconf/blob/master/backupStatus.png" width="420" float="right">

## Alerting System for Backup & Restore
In case when backup fails or after restore the service instance is not running, email based alerts are raised.
<p align="center">
  <img src="https://github.com/akashkumar58/pgconf/blob/master/backupAlert.png" width="600"/>
</p>
