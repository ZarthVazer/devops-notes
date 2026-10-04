# Remote state and locking on AWS

## Why

Local `terraform.tfstate` breaks as soon as two people (or a person and CI) run
Terraform: states diverge and resources get duplicated or destroyed. Remote state
gives one source of truth; locking prevents two `apply` runs at the same time.

## Setup

```hcl
terraform {
  backend "s3" {
    bucket       = "my-company-terraform-state"
    key          = "prod/network/terraform.tfstate"
    region       = "eu-central-1"
    encrypt      = true
    use_lockfile = true   # S3-native locking (Terraform >= 1.11)
  }
}
```

Older setups use a DynamoDB table for locking instead (`dynamodb_table = "terraform-locks"`
with a `LockID` string hash key). New projects can use S3-native locking.

The state bucket itself should have:

- **versioning enabled** — lets you recover a previous state after a bad apply
- **encryption** and **public access block**
- access limited to the CI role and a few admins

## One state per environment and component

```
prod/network/terraform.tfstate
prod/eks/terraform.tfstate
dev/network/terraform.tfstate
```

Smaller states mean faster plans and a smaller blast radius when something goes wrong.

## Stuck lock

If a run was killed and the lock remains:

```bash
terraform force-unlock <LOCK_ID>
```

Only do this after confirming nobody else is actually running Terraform.
