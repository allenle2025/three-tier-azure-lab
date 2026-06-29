# Three-Tier Azure Network Segmentation Lab

## Overview

A production-grade zero-trust network architecture demonstrating deny-by-default NSG segmentation across three tiers (web, app, database). This lab reflects secure network patterns required by federal contractors and government agencies.

**Key Achievement:** Validated end-to-end SSH jumpbox chain with complete traffic isolation—public internet can reach only the web tier; backend tiers are entirely inaccessible from outside.

## Architecture
Internet → Public IP → vm-web (web tier) → vm-app (app tier) → vm-db (database tier)
                          ↓                      ↓                    ↓
                       nsg-web                nsg-app              nsg-db
                    (allow SSH)            (allow app->db)      (allow db-only)

**Three Subnets:**
- `subnet-web` (10.0.1.0/24) — Internet-facing layer with public IP
- `subnet-app` (10.0.2.0/24) — Application logic, reachable only from web tier
- `subnet-db` (10.0.3.0/24) — Database, reachable only from app tier

## Security Rules (Deny-by-Default)

### nsg-web
- ✅ Allow HTTP (80), HTTPS (443), SSH (22) from Internet
- ✅ Allow outbound to app tier (port varies by app)
- ❌ Deny all other inbound
- ❌ Deny direct database access

### nsg-app
- ✅ Allow inbound from web tier only
- ✅ Allow outbound to database tier (port 1433 for SQL, or custom)
- ❌ Deny direct Internet access
- ❌ Deny public ingress

### nsg-db
- ✅ Allow inbound from app tier only (port 1433 for SQL)
- ✅ Allow outbound to VirtualNetwork for replication
- ❌ Deny all direct inbound
- ❌ Deny public access entirely

## Validation & Testing

All paths validated end-to-end:

**✅ Allowed Traffic:**
- Internet → vm-web (SSH/HTTP/HTTPS)
- vm-web → vm-app (private chain)
- vm-app → vm-db (private chain)

**❌ Blocked Traffic (Negative Tests):**
- Internet → vm-db (connection timeout)
- vm-web → vm-db directly (connection timeout, must go through app tier)

See [validation screenshots](./validation/) for SSH session proof.

## Resources Deployed

- Virtual Network: `vnet-lab` (10.0.0.0/16)
- 3x Ubuntu 24.04 LTS VMs (Ubuntu chosen for cloud-native alignment)
- 3x Network Security Groups (nsg-web, nsg-app, nsg-db)
- 3x SSH Key Pairs (stored securely in user .ssh directory)
- 1x Public IP (attached to vm-web only)

## Next Steps

**Planned:** Terraform IaC version with automated deployment, state management, and reusability for multi-environment rollout.
