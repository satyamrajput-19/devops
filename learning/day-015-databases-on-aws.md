# Day 15 — Databases on AWS

## Learning objectives
- Explain SQL vs. NoSQL and choose a database based on workload and access patterns.
- Describe Amazon RDS Multi-AZ deployments, read replicas, backups, and recovery.
- Explain DynamoDB tables, primary keys, partition keys, sort keys, and capacity modes.
- Explain ElastiCache and when caching helps.
- Practice safe database networking and least-privilege access.

## 1. SQL vs. NoSQL

### Relational / SQL
Relational databases organize data into tables with defined relationships and typically use SQL. They are a strong fit for structured data, joins, constraints, and transactional workloads.

Examples on AWS: Amazon RDS engines such as PostgreSQL and MySQL, and Amazon Aurora.

### NoSQL
NoSQL covers multiple data models, including key-value and document databases. Design is often driven by the queries and access patterns the application needs.

Example on AWS: Amazon DynamoDB, a managed key-value/document database.

| Question | Relational database | DynamoDB |
|---|---|---|
| Data model | Tables, rows, relationships | Items with attributes |
| Query style | SQL, joins, filters | Key-based access patterns and supported query operations |
| Schema | Defined table structure | Flexible item attributes, with key design still important |
| Common fit | Relationships and transactional queries | High-scale key-value/document access patterns |

Do not choose based on “SQL is old” or “NoSQL is faster.” Choose based on consistency, relationships, query patterns, scale, operational needs, and cost.

## 2. Amazon RDS

Amazon Relational Database Service (RDS) manages common database operations such as provisioning, patching options, backups, monitoring, and recovery features for supported engines.

Key terms:
- **DB instance:** Compute and storage environment for a database.
- **DB subnet group:** Subnets RDS can use in a VPC.
- **Security group:** Network rules controlling which clients can reach the database port.
- **Automated backups:** Support point-in-time recovery within the configured retention period, subject to engine and service constraints.
- **Manual snapshots:** User-initiated snapshots retained until deleted.
- **Encryption:** Use encryption at rest and TLS in transit where supported and configured.

### Multi-AZ vs. read replicas

**Multi-AZ deployment**
- Primarily a high-availability and failover approach.
- RDS maintains a standby in another Availability Zone for supported configurations.
- Automatic failover can reduce downtime, but it is not zero downtime.
- A standby in a traditional Multi-AZ DB instance deployment is not used to serve ordinary read traffic.

**Read replica**
- Replicates data from a source database to scale read-heavy workloads.
- Replication is generally asynchronous, so replica lag and stale reads are possible.
- It is not a substitute for a tested backup and recovery plan.
- Availability, promotion, and cross-Region options depend on the database engine and configuration.

Remember: **Multi-AZ = availability; read replica = read scaling** (with some engine-specific exceptions and options).

## 3. Backups and recovery

Understand the difference:
- Automated backups support point-in-time recovery within the retention period.
- Manual snapshots are retained until you delete them.
- A backup is useful only if restoration has been tested and the recovery time and recovery point objectives are understood.

Lab checklist:
1. Review backup retention and maintenance settings for an RDS lab instance.
2. Create a manual snapshot if your lab budget permits.
3. Document the restore workflow without exposing credentials.
4. Record what a restore would mean for endpoint changes, application configuration, and recovery time.
5. Delete resources and snapshots you no longer need, while keeping any snapshot you intentionally want to retain.

## 4. Amazon DynamoDB

Core concepts:
- **Table:** Collection of items.
- **Item:** A record in a table.
- **Attribute:** A field on an item.
- **Partition key:** Determines the partitioning of data and is essential to access-pattern design.
- **Sort key:** Optional key used to organize related items and support range queries.
- **Capacity mode:** On-demand or provisioned, depending on workload and operational preference.
- **Indexes:** Secondary indexes support additional query patterns, with trade-offs in cost and data modeling.

Design tables around the access patterns you need. Avoid assuming that a relational schema can be copied directly into DynamoDB without considering keys and query requirements.

## 5. Amazon ElastiCache

ElastiCache provides managed in-memory caching engines, including Redis OSS and Memcached options depending on current service support.

Common uses:
- Cache frequently read data to reduce database load.
- Store short-lived sessions or application state where the chosen engine and durability model are appropriate.
- Cache computed results that are expensive to regenerate.

Important trade-offs:
- Cached data can become stale.
- Cache invalidation and TTLs must be designed.
- A cache is not automatically the source of truth.
- Authentication, network isolation, encryption, and access controls still matter.

Typical cache-aside flow:
1. Application checks the cache.
2. On a hit, return the cached value.
3. On a miss, read from the database.
4. Store the result in the cache with a suitable TTL.
5. Invalidate or update the cache when the underlying data changes.

## 6. Hands-on lab plan

Use a disposable AWS lab account and set a budget alert first. RDS and ElastiCache can incur ongoing charges.

### Lab A — RDS networking and security
- Inspect a VPC and private subnets.
- Review an RDS DB subnet group.
- Create or inspect a database security group that allows the database port only from the application security group.
- Do not expose a production database publicly.
- Connect from an authorized client inside the VPC or through an approved secure path.
- Do not put passwords, connection strings, access keys, or secret values in GitHub.

### Lab B — RDS resilience concepts
- Compare Single-AZ and Multi-AZ configuration options.
- Explain how failover changes client connection behavior.
- Compare automated backups, point-in-time recovery, and manual snapshots.
- If creating resources, use a small lab configuration and delete it after testing.

### Lab C — DynamoDB modeling
- Draft a table for a simple shopping cart.
- Identify partition key, optional sort key, and at least three access patterns.
- Explain why each access pattern is supported by your key design.
- Compare on-demand and provisioned capacity at a conceptual level.

### Lab D — Caching design
- Draw a cache-aside request flow.
- Choose a TTL for a non-critical example such as product catalogue data.
- Explain what happens on cache miss, stale data, cache failure, and database failure.

## 7. Interview questions

1. What is the difference between RDS and DynamoDB?
2. What is the difference between Multi-AZ and a read replica?
3. Why can a read replica return stale data?
4. What is point-in-time recovery?
5. How is a manual snapshot different from an automated backup?
6. Why should an RDS database usually be placed in private subnets?
7. How should a database security group be configured for an application?
8. What is a DynamoDB partition key, and why does key design matter?
9. When would you use a sort key or secondary index?
10. What problem does ElastiCache solve?
11. What is cache invalidation, and why is it difficult?
12. How would you recover a database after accidental data deletion?

## 8. Practical checkpoint

Before marking Day 15 complete, be able to:
- Explain Multi-AZ vs. read replicas without notes.
- Describe a safe RDS network design.
- Compare automated backups and snapshots.
- Propose DynamoDB keys for a stated access pattern.
- Draw a cache-aside flow and explain stale-data handling.
- Document the lab, screenshots/diagrams where useful, troubleshooting, and cleanup steps.

## 9. GitHub evidence

Add your notes to this file and include:
- A simple RDS private-network diagram.
- A Multi-AZ vs. read-replica comparison.
- A DynamoDB table/key design for one use case.
- A cache-aside diagram.
- Lab observations and cleanup confirmation.

## Cost and security reminders
- RDS, ElastiCache, snapshots, data transfer, and related networking can incur charges.
- Review current AWS pricing and set a budget alert before starting.
- Delete disposable resources after the lab; check snapshots separately.
- Never commit credentials or real customer data.

## Next session
Day 16 — CloudFormation, CloudWatch, and Serverless: infrastructure as code, metrics, logs, alarms, SNS/SQS, Lambda, and API Gateway.
