# MediaConnect Bridge

The MediaConnect Bridge resource allows you to create AWS Elemental MediaConnect bridges for connecting cloud and on-premises video workflows.

## Overview

A MediaConnect bridge enables you to send content between your on-premises equipment and MediaConnect flows in the cloud. There are two types of bridges:

- **Egress Gateway Bridge**: Sends content from MediaConnect flows to your on-premises equipment (cloud-to-ground)
- **Ingress Gateway Bridge**: Sends content from your on-premises equipment to MediaConnect flows (ground-to-cloud)

## API Reference

### Resource Types

- **Composite Resource**: `XBridge` (cluster-scoped)
- **Claim**: `Bridge` (namespace-scoped)
- **API Version**: `mediaconnect.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `placementArn` | string | Yes | The bridge placement Amazon Resource Number (ARN) |
| `sources` | array | Yes | The sources that you want to add to this bridge (0-2 items) |
| `outputs` | array | No | The outputs that you want to add to this bridge (0-2 items) |
| `egressGatewayBridge` | object | No | Configuration for cloud-to-ground bridge |
| `ingressGatewayBridge` | object | No | Configuration for ground-to-cloud bridge |
| `sourceFailoverConfig` | object | No | Settings for source failover |
| `region` | string | No | AWS region where the bridge will be created |
| `tags` | object | No | Key-value pairs to associate with the bridge |

### Sources Configuration

Each source can be either a `FlowSource` or `NetworkSource`:

#### FlowSource
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `FlowArn` | string | Yes | The ARN of the cloud flow used as a source |
| `Name` | string | Yes | The name of the flow source |
| `FlowVpcInterfaceAttachment` | object | No | VPC interface attachment configuration |

#### NetworkSource
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `MulticastIp` | string | Yes | The network source multicast IP |
| `Name` | string | Yes | The name of the network source |
| `NetworkName` | string | Yes | The network source's gateway network name |
| `Port` | integer | Yes | The network source port (1-65535) |
| `Protocol` | string | Yes | The network source protocol (`rtp-fec`, `rtp`, `udp`) |

### Outputs Configuration

#### NetworkOutput
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | string | Yes | The network output name |
| `NetworkOutput.IpAddress` | string | Yes | The network output IP address |
| `NetworkOutput.Name` | string | Yes | The network output name |
| `NetworkOutput.NetworkName` | string | Yes | The network output's gateway network name |
| `NetworkOutput.Port` | integer | Yes | The network output's port (1-65535) |
| `NetworkOutput.Protocol` | string | Yes | The network output protocol (`rtp-fec`, `rtp`, `udp`) |
| `NetworkOutput.Ttl` | integer | Yes | The network output TTL (1-255) |

### Gateway Bridge Configuration

#### EgressGatewayBridge
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `MaxBitrate` | integer | Yes | Maximum expected bitrate (in bps) of the egress bridge |

#### IngressGatewayBridge
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `MaxBitrate` | integer | Yes | Maximum expected bitrate (in bps) of the ingress bridge |
| `MaxOutputs` | integer | Yes | Maximum number of outputs on the ingress bridge |

### Source Failover Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `FailoverMode` | string | Yes | The type of failover (`FAILOVER`) |
| `State` | string | No | State of source failover (`ENABLED`, `DISABLED`) |
| `SourcePriority` | object | No | Priority assignment configuration |

## Examples

### Basic Egress Gateway Bridge

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Bridge
metadata:
  name: my-egress-bridge
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    placementArn: arn:aws:mediaconnect:us-east-1:123456789012:bridge-placement/example-placement
    sources:
      - FlowSource:
          FlowArn: arn:aws:mediaconnect:us-east-1:123456789012:flow/example-flow
          Name: primary-source
    outputs:
      - Name: primary-output
        NetworkOutput:
          IpAddress: 192.168.1.100
          Name: primary-output
          NetworkName: example-network
          Port: 5000
          Protocol: rtp
          Ttl: 64
    egressGatewayBridge:
      MaxBitrate: 50000000
    tags:
      Environment: production
      Application: video-streaming
```

### Ingress Gateway Bridge with Network Source

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Bridge
metadata:
  name: my-ingress-bridge
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    placementArn: arn:aws:mediaconnect:us-west-2:123456789012:bridge-placement/example-placement
    sources:
      - NetworkSource:
          MulticastIp: 224.1.1.1
          Name: camera-feed
          NetworkName: studio-network
          Port: 5004
          Protocol: rtp-fec
    ingressGatewayBridge:
      MaxBitrate: 100000000
      MaxOutputs: 2
    tags:
      Environment: production
      Source: studio-camera
```

### Bridge with Source Failover

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Bridge
metadata:
  name: my-failover-bridge
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    placementArn: arn:aws:mediaconnect:us-east-1:123456789012:bridge-placement/example-placement
    sources:
      - FlowSource:
          FlowArn: arn:aws:mediaconnect:us-east-1:123456789012:flow/primary-flow
          Name: primary-source
      - FlowSource:
          FlowArn: arn:aws:mediaconnect:us-east-1:123456789012:flow/backup-flow
          Name: backup-source
    sourceFailoverConfig:
      FailoverMode: FAILOVER
      State: ENABLED
      SourcePriority:
        PrimarySource: primary-source
        RecoveryWindow: 3000
    egressGatewayBridge:
      MaxBitrate: 50000000
```

## Status

The bridge status includes information about the AWS resource state and any connection details.

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

## AWS Documentation

For more detailed information about MediaConnect bridges, refer to the official AWS documentation:

- [AWS MediaConnect Bridges](https://docs.aws.amazon.com/mediaconnect/latest/ug/bridges.html)
- [AWS Cloud Control API - MediaConnect Bridge](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations.html)
- [AWS::MediaConnect::Bridge CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-mediaconnect-bridge.html)

## Notes

- Bridge placement ARN must be created beforehand in your AWS account
- Network sources and outputs require proper network configuration
- Source failover is only available when you have multiple sources
- Maximum of 2 sources and 2 outputs per bridge
- Egress and ingress gateway bridge configurations are mutually exclusive
