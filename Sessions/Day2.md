# Day 2 Session Notes

## Table of Contents
1. [Compute Engines & Dataproc](#compute-engines--dataproc)
2. [Cloud Identity and Access Management](#cloud-identity-and-access-management)
3. [Compute](#compute)
4. [Dataproc](#dataproc)
5. [Dataflow](#dataflow)

---

## Cloud Identity and Access Management

### Core Concept
> Who can do what on which resource

### Granular Access
Granular access control (fine-grained access control) is a security mechanism that allows organizations to manage and restrict access to their resources, systems, or data at a highly detailed and specific level.

### Types of Roles

1. **Basic Roles**
   - Most limited form of GCP roles
   - Includes owners, editors, and viewers

2. **Predefined Roles**
   - Finer-grain access to specific services
   - Service-specific permissions

3. **Custom Roles**
   - Organization-specific permissions
   - Meets specific needs
   - Finer-grain access control

### Basic Role Types
- **Owner**: Can invite/remove members, delete projects
- **Editor**: Can modify resources
- **Viewer**: Read-only access
- **Billing Administrator**: Manages billing

---

## Compute

### Sole-tenant Nodes
- Dedicated to single customer
- Features:
  - Dedicated hardware
  - Full access to host resources (10% premium)

### Managed Instance Groups
Running VMs at Scale
- High availability through:
  - Autoscaling
  - Load balancing
  - Auto-healing

### Instance Types
1. **Stateful vs Stateless**
   - Different persistence requirements
   - Different scaling approaches

2. **Instance Groups**
   - Unmanaged
   - Managed

### MIG-Autoscaling (Managed Instance Group)
Scaling based on:
- CPU Utilization
- External HTTPS Capacity
- Cloud Monitoring Metrics
- Schedules

---

## Dataproc
Managed Spark and Hadoop service
- Simplified big data processing
- Integrated with other Google Cloud services

---

## Dataflow
Fully-managed ETL tool
- Streaming data processing
- Batch data processing
- Real-time analytics
