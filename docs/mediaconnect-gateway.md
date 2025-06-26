# MediaConnect Gateway

The MediaConnect Gateway resource allows you to create AWS Elemental MediaConnect gateways for connecting on-premises equipment to MediaConnect flows in the cloud.

## Overview

AWS Elemental MediaConnect gateways provide a bridge between your on-premises video equipment and MediaConnect flows in the AWS cloud. Gateways enable secure, reliable transport of live video content over IP networks, allowing you to extend your on-premises workflows to the cloud seamlessly.

## API Reference

### Resource Types

- **Composite Resource**: `XGateway` (cluster-scoped)
- **Claim**: `Gateway` (namespace-scoped)
- **API Version**: `mediaconnect.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `egressCidrBlocks` | array | Yes | IP address ranges allowed to contribute content or initiate output requests |
| `networks` | array | Yes | List of networks in the gateway (1-4 networks) |
| `region` | string | No | AWS region where the gateway will be created |
| `tags` | object | No | Key-value pairs to associate with the gateway |

### Egress CIDR Blocks

An array of CIDR blocks specifying IP address ranges that are allowed to contribute content or initiate output requests for flows communicating with this gateway.

- **Format**: CIDR notation (e.g., `10.0.0.0/16`)
- **Pattern**: `^([0-9]{1,3}\.){3}[0-9]{1,3}/([0-9]|[1-2][0-9]|3[0-2])$`
- **Minimum Items**: 1

### Networks Configuration

Each network in the gateway must specify:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | string | Yes | Name of the network (must be unique within the gateway) |
| `CidrBlock` | string | Yes | IP address range for this network in CIDR format |

**Constraints:**
- Minimum: 1 network
- Maximum: 4 networks
- CIDR block pattern: `^([0-9]{1,3}\.){3}[0-9]{1,3}/([0-9]|[1-2][0-9]|3[0-2])$`

## Examples

### Basic Gateway

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Gateway
metadata:
  name: studio-gateway
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    egressCidrBlocks:
      - "10.0.0.0/16"
      - "192.168.1.0/24"
    networks:
      - Name: studio-network
        CidrBlock: "10.0.100.0/24"
    tags:
      Environment: production
      Location: studio-a
```

### Multi-Network Gateway

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Gateway
metadata:
  name: broadcast-center-gateway
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    egressCidrBlocks:
      - "172.16.0.0/12"
      - "10.0.0.0/8"
    networks:
      - Name: control-room-network
        CidrBlock: "172.16.1.0/24"
      - Name: studio-1-network
        CidrBlock: "172.16.2.0/24"
      - Name: studio-2-network
        CidrBlock: "172.16.3.0/24"
      - Name: equipment-network
        CidrBlock: "172.16.10.0/24"
    tags:
      Environment: production
      Facility: broadcast-center
      Department: engineering
```

### Development Gateway

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Gateway
metadata:
  name: dev-gateway
  namespace: development
spec:
  providerConfigRef:
    name: aws-dev-provider-config
  forProvider:
    region: us-east-1
    egressCidrBlocks:
      - "192.168.0.0/16"
    networks:
      - Name: dev-network
        CidrBlock: "192.168.100.0/24"
    tags:
      Environment: development
      Purpose: testing
```

### Enterprise Gateway with Multiple Locations

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Gateway
metadata:
  name: enterprise-gateway
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    egressCidrBlocks:
      - "10.0.0.0/8"
      - "172.16.0.0/12"
      - "192.168.0.0/16"
    networks:
      - Name: headquarters-network
        CidrBlock: "10.1.0.0/16"
      - Name: branch-office-network
        CidrBlock: "10.2.0.0/16"
      - Name: remote-studio-network
        CidrBlock: "10.3.0.0/16"
      - Name: backup-facility-network
        CidrBlock: "10.4.0.0/16"
    tags:
      Environment: production
      Organization: enterprise
      Region: east-coast
      CostCenter: "12345"
```

## Status

The gateway status includes information about the current state of the gateway and AWS resource details.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: Available
  state: ACTIVE
  atProvider:
    # AWS-specific resource information
    properties: {}
```

### Gateway States

The gateway can be in one of the following states:

- `CREATING`: Gateway is being created
- `ACTIVE`: Gateway is operational and ready for use
- `UPDATING`: Gateway configuration is being updated
- `ERROR`: Gateway encountered an error
- `DELETING`: Gateway is being deleted
- `DELETED`: Gateway has been successfully deleted

## Use Cases

### Studio-to-Cloud Workflows
Gateways enable studios to send live video content to MediaConnect flows for cloud-based processing and distribution.

### Multi-Location Broadcasting
Connect multiple studio locations through a single gateway, allowing content sharing and backup workflows.

### Hybrid Cloud Architectures
Integrate on-premises broadcast equipment with cloud-based MediaConnect flows for scalable content distribution.

### Disaster Recovery
Establish redundant pathways for critical video content through multiple gateway networks.

## AWS Documentation

For more detailed information about MediaConnect gateways, refer to the official AWS documentation:

- [AWS MediaConnect Gateways](https://docs.aws.amazon.com/mediaconnect/latest/ug/gateways.html)
- [AWS Cloud Control API - MediaConnect Gateway](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations.html)
- [AWS::MediaConnect::Gateway CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-mediaconnect-gateway.html)

## Notes

- **Network Planning**: Carefully plan your network CIDR blocks to avoid conflicts with existing infrastructure
- **Security**: Egress CIDR blocks should be as restrictive as possible while allowing necessary traffic
- **Scalability**: You can configure up to 4 networks per gateway
- **Regional Deployment**: Gateways are region-specific resources
- **State Monitoring**: Monitor the gateway state to ensure operational readiness
- **Network Isolation**: Each network within a gateway provides logical isolation for different types of traffic
- **IP Address Management**: Ensure CIDR blocks don't overlap with your existing network infrastructure
- **Firewall Configuration**: Configure your on-premises firewalls to allow traffic to/from the specified CIDR blocks
