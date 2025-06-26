# MediaLive Network

The MediaLive Network resource allows you to create AWS Elemental MediaLive networks for managing IP address pools and routing configurations in live video workflows.

## Overview

AWS Elemental MediaLive networks define IP address pools and routing configurations that can be used by MediaLive channels and other resources. Networks provide logical separation and organization of IP resources, enabling better network management and traffic routing for live video processing workflows.

## API Reference

### Resource Types

- **Composite Resource**: `XNetwork` (cluster-scoped)
- **Claim**: `Network` (namespace-scoped)
- **API Version**: `medialive.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ipPools` | array | Yes | List of IP address CIDR pools for the network (minimum 1) |
| `routes` | array | No | Routes for the network |
| `region` | string | No | AWS region where the network will be created |
| `tags` | object | No | Key-value pairs to associate with the network |

### IP Pools Configuration

Each IP pool must specify:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Cidr` | string | Yes | The CIDR block for the IP pool |

**Constraints:**
- **Pattern**: `^([0-9]{1,3}\.){3}[0-9]{1,3}/([0-9]|[1-2][0-9]|3[0-2])$`
- **Minimum Items**: 1

### Routes Configuration

Each route can specify:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Cidr` | string | No | The CIDR block for the route |
| `Gateway` | string | No | The gateway IP address for the route |

**Constraints:**
- **CIDR Pattern**: `^([0-9]{1,3}\.){3}[0-9]{1,3}/([0-9]|[1-2][0-9]|3[0-2])$`

## Examples

### Basic Network

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: basic-network
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    ipPools:
      - Cidr: "10.0.0.0/24"
    tags:
      Environment: production
      Purpose: live-streaming
```

### Network with Multiple IP Pools

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: multi-pool-network
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    ipPools:
      - Cidr: "192.168.1.0/24"
      - Cidr: "192.168.2.0/24"
      - Cidr: "192.168.3.0/24"
    tags:
      Environment: production
      Location: west-coast
      Capacity: high
```

### Network with Routes

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: routed-network
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    ipPools:
      - Cidr: "172.16.0.0/16"
    routes:
      - Cidr: "10.0.0.0/8"
        Gateway: "172.16.1.1"
      - Cidr: "192.168.0.0/16"
        Gateway: "172.16.1.1"
    tags:
      Environment: production
      Network: corporate
      Routing: enabled
```

### Studio Network Configuration

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: studio-network
  namespace: broadcast
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    ipPools:
      - Cidr: "10.100.0.0/22"
      - Cidr: "10.100.4.0/22"
    routes:
      - Cidr: "10.0.0.0/8"
        Gateway: "10.100.0.1"
      - Cidr: "172.16.0.0/12"
        Gateway: "10.100.0.1"
    tags:
      Environment: production
      Facility: studio-a
      Department: broadcast-engineering
      Redundancy: enabled
```

### Development Network

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: dev-network
  namespace: development
spec:
  providerConfigRef:
    name: aws-dev-provider-config
  forProvider:
    region: us-west-2
    ipPools:
      - Cidr: "192.168.100.0/24"
    tags:
      Environment: development
      Purpose: testing
      CostOptimized: "true"
```

### Multi-Region Network

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: global-network
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    ipPools:
      - Cidr: "10.10.0.0/16"
      - Cidr: "10.20.0.0/16"
    routes:
      - Cidr: "10.30.0.0/16"
        Gateway: "10.10.1.1"
      - Cidr: "10.40.0.0/16"
        Gateway: "10.20.1.1"
    tags:
      Environment: production
      Scope: global
      Region: primary
      Failover: supported
```

### Broadcast Center Network

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: broadcast-center-network
  namespace: broadcast
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    ipPools:
      - Cidr: "172.20.0.0/20"
      - Cidr: "172.20.16.0/20"
      - Cidr: "172.20.32.0/20"
    routes:
      - Cidr: "172.16.0.0/12"
        Gateway: "172.20.0.1"
      - Cidr: "10.0.0.0/8"
        Gateway: "172.20.0.1"
      - Cidr: "192.168.0.0/16"
        Gateway: "172.20.16.1"
    tags:
      Environment: production
      Facility: broadcast-center
      Capacity: enterprise
      SLA: premium
```

## Status

The network status includes information about the AWS resource state.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: Available
  atProvider:
    # AWS-specific resource information
    properties: {}
```

## IP Pool Planning

### Common CIDR Blocks

**Private IP Ranges (RFC 1918):**
- `10.0.0.0/8` - Class A private range
- `172.16.0.0/12` - Class B private range
- `192.168.0.0/16` - Class C private range

**Subnet Sizing Guidelines:**
- `/24` - 254 usable IPs (small networks)
- `/22` - 1,022 usable IPs (medium networks)
- `/20` - 4,094 usable IPs (large networks)
- `/16` - 65,534 usable IPs (very large networks)

### Best Practices

1. **Avoid Overlaps**: Ensure CIDR blocks don't overlap with existing networks
2. **Room for Growth**: Plan for future expansion when sizing subnets
3. **Logical Separation**: Use different ranges for different purposes/environments
4. **Documentation**: Document IP assignments and network purposes

## Use Cases

### Live Event Production
Create dedicated networks for live event workflows with isolated IP pools for different production elements.

### Broadcast Facility Management
Organize studio networks with separate IP pools for control rooms, equipment racks, and transmission systems.

### Multi-Tenant Environments
Provide network isolation for different customers or departments sharing MediaLive infrastructure.

### Development and Testing
Separate development networks from production with dedicated IP ranges.

### Disaster Recovery
Establish backup networks with pre-configured IP pools for failover scenarios.

## Routing Configuration

### Static Routes
Configure static routes to direct traffic between different network segments or to external gateways.

### Gateway Configuration
Specify gateway IP addresses for routing traffic outside the local network segments.

### Network Segmentation
Use routes to implement network segmentation and control traffic flow between different parts of your infrastructure.

## AWS Documentation

For more detailed information about MediaLive networks, refer to the official AWS documentation:

- [AWS MediaLive Networks](https://docs.aws.amazon.com/medialive/latest/ug/networks.html)
- [AWS Cloud Control API - MediaLive Network](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations.html)
- [AWS::MediaLive::Network CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-medialive-network.html)

## Notes

- **IP Pool Requirement**: At least one IP pool must be specified
- **CIDR Validation**: All CIDR blocks must follow valid IPv4 CIDR notation
- **Regional Scope**: Networks are region-specific resources
- **Route Dependencies**: Gateway IPs should be within the network's IP pools
- **Network Planning**: Plan IP addressing carefully to avoid conflicts
- **Integration**: Networks can be referenced by MediaLive channels and other resources
- **Monitoring**: Monitor network utilization to ensure adequate IP pool capacity
- **Security**: Consider security group and NACL rules when planning network access
