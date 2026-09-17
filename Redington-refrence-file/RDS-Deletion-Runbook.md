# Runbook: Deletion of Stopped RDS Instances

**Environment:** AWS RDS (ap-south-1)
**Owner:** Muthu S S
**Last Updated:** 15-Sep-2026

## Purpose
Safely delete stopped/unused RDS instances identified in the Databases console, with a mandatory backup step for the production database.

## Scope

| DB Identifier | Type | Backup Required |
|---|---|---|
| javelin-production-live | RDS Instance (MySQL Community) | Yes |
| migration-test-aurora-cluster (incl. migration-test-aurora) | Aurora MySQL Cluster | No |
| testing-rds-new | RDS Instance (MySQL Community) | No |

## Prerequisites
- IAM permissions: `rds:DeleteDBInstance`, `rds:DeleteDBCluster`, `rds:CreateDBSnapshot`, `rds:DescribeDBInstances`
- Confirm no active application/service is still dependent on these databases
- Confirm with stakeholders before touching `javelin-production-live` (production)

---

## Step 1 — Take a Manual Snapshot of Production DB

1. Open AWS Console → RDS → Databases
2. Select `javelin-production-live`
3. Click **Actions** → **Take snapshot**
4. Snapshot name: `javelin-production-live-final-backup-YYYYMMDD`
5. Wait until snapshot status = **Available** (check under Snapshots → Manual)

**Checkpoint:** Do not proceed to Step 2 until the snapshot shows Available.

## Step 2 — Delete javelin-production-live

1. Select `javelin-production-live`
2. Click **Actions** → **Delete**
3. In the confirmation dialog:
   - Uncheck "Create final snapshot" (already taken manually)
   - Uncheck "Retain automated backups" (unless policy requires retention)
   - Type the required confirmation text
4. Click **Delete**
5. Verify instance disappears from the active Databases list

## Step 3 — Delete migration-test-aurora-cluster

> Note: `migration-test-aurora` is a writer instance inside this cluster. Deleting the cluster removes the instance with it.

1. Select `migration-test-aurora-cluster` (the cluster, not the instance)
2. Click **Actions** → **Delete**
3. Uncheck "Create final snapshot" (test resource, not required)
4. Type the confirmation text
5. Click **Delete**
6. Verify both the cluster and `migration-test-aurora` instance are removed

## Step 4 — Delete testing-rds-new

1. Select `testing-rds-new`
2. Click **Actions** → **Delete**
3. Uncheck "Create final snapshot" and "Retain automated backups"
4. Type the confirmation text
5. Click **Delete**
6. Verify it disappears from the Databases list

## Step 5 — Post-Deletion Verification

1. Refresh RDS → Databases and confirm the count dropped by 3 (8 → 5)
2. Confirm remaining databases (Available ones) are unaffected
3. Confirm the manual snapshot for `javelin-production-live` is visible under Snapshots
4. Notify stakeholders that the deletion activity is complete

## Rollback / Recovery
- If `javelin-production-live` needs to be restored, use **RDS → Snapshots → Manual → Restore Snapshot** with the snapshot taken in Step 1.
- Test/migration resources (`migration-test-aurora`, `testing-rds-new`) have no snapshot and cannot be recovered once deleted — confirm before proceeding if this changes.

## Sign-off

---
*End of runbook*
