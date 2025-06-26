# MediaConnect Flow

The MediaConnect Flow resource allows you to create AWS Elemental MediaConnect flows for transporting live video content over IP networks.

## Overview

AWS Elemental MediaConnect flows are used to securely transport live video content between locations. Flows can ingest content from various sources and distribute it to multiple destinations with enterprise-grade security and reliability.

## API Reference

### Resource Types

- **Composite Resource**: `XFlow` (cluster-scoped)
- **Claim**: `Flow` (namespace-scoped)
- **API Version**: `mediaconnect.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `source` | object | Yes | The source configuration for the flow |
| `availabilityZone` | string | No | The Availability Zone for the flow |
| `flowSize` | string | No | Processing capacity (`MEDIUM`, `LARGE`) |
| `sourceFailoverConfig` | object | No | Settings for source failover |
| `mediaStreams` | array | No | Media streams associated with the flow |
| `vpcInterfaces` | array | No | VPC interfaces for the flow (max 3) |
| `maintenance` | object | No | Maintenance settings for the flow |
| `region` | string | No | AWS region where the flow will be created |
| `tags` | object | No | Key-value pairs to associate with the flow |

### Source Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | string | No | The name of the source |
| `Protocol` | string | No | Protocol (`zixi-push`, `rtp-fec`, `rtp`, `rist`, `st2110-jpegxs`, `cdi`, `srt-listener`, `srt-caller`, `fujitsu-qos`) |
| `Description` | string | No | A description for the source |
| `IngestPort` | integer | No | Port for receiving content (1-65535) |
| `MaxBitrate` | integer | No | Smoothing max bitrate for RIST, RTP, and RTP-FEC |
| `MaxLatency` | integer | No | Maximum latency in milliseconds (1-15000) |
| `MinLatency` | integer | No | Minimum latency for SRT-based streams (1-15000) |
| `SenderIpAddress` | string | No | IP address for sender communication |
| `SenderControlPort` | integer | No | Port for outbound requests (1-65535) |
| `StreamId` | string | No | Stream ID for this transport |
| `VpcInterfaceName` | string | No | Name of VPC interface to use |
| `WhitelistCidr` | string | No | IP range allowed to contribute content |
| `Decryption` | object | No | Encryption configuration |
| `EntitlementArn` | string | No | ARN of entitlement for cross-account access |
| `GatewayBridgeSource` | object | No | Configuration for bridge sources |

### Source Failover Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `FailoverMode` | string | No | Type of failover (`MERGE`, `FAILOVER`) |
| `RecoveryWindow` | integer | No | Recovery window time (100-15000ms) |
| `SourcePriority` | object | No | Priority assignment for sources |
| `State` | string | No | Failover state (`ENABLED`, `DISABLED`) |

### Media Streams Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `MediaStreamId` | integer | Yes | Unique identifier for the media stream |
| `MediaStreamName` | string | Yes | Name to distinguish the media stream |
| `MediaStreamType` | string | Yes | Type of media (`video`, `audio`, `ancillary-data`) |
| `Attributes` | object | No | Sample rate, bitrate, and other attributes |
| `ClockRate` | integer | No | Sample rate in Hz |
| `Description` | string | No | Description of the media stream |
| `Fmt` | integer | No | Format type number (0-127) |
| `VideoFormat` | string | No | Video resolution (`2160p`, `1080p`, `1080i`, `720p`, `480p`) |

### VPC Interfaces Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | string | Yes | Name of the VPC interface |
| `RoleArn` | string | Yes | Role ARN for MediaConnect to create ENI |
| `SecurityGroupIds` | array | Yes | Security Group IDs for the ENI |
| `SubnetId` | string | Yes | Subnet ID (must be in flow's AZ) |
| `NetworkInterfaceType` | string | No | Network interface type (`ena`, `efa`) |

### Maintenance Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `MaintenanceDay` | string | Yes | Day of week for maintenance |
| `MaintenanceStartHour` | string | Yes | UTC time for maintenance (HH:MM format) |

## Examples

### Basic Flow with Zixi Push Source

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Flow
metadata:
  name: basic-zixi-flow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    availabilityZone: us-east-1a
    source:
      Name: zixi-source
      Protocol: zixi-push
      Description: "Primary video feed from studio"
      IngestPort: 2088
      WhitelistCidr: "10.0.0.0/16"
    tags:
      Environment: production
      Application: live-streaming
```

### Flow with SRT Source and Encryption

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Flow
metadata:
  name: srt-encrypted-flow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    flowSize: LARGE
    source:
      Name: srt-source
      Protocol: srt-listener
      Description: "SRT feed with encryption"
      IngestPort: 9001
      MaxLatency: 2000
      MinLatency: 120
      Decryption:
        Algorithm: aes256
        KeyType: static-key
        SecretArn: arn:aws:secretsmanager:us-west-2:123456789012:secret:srt-key
        RoleArn: arn:aws:iam::123456789012:role/MediaConnectRole
    tags:
      Environment: production
      Protocol: srt
```

### Flow with Source Failover

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Flow
metadata:
  name: failover-flow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    source:
      Name: primary-source
      Protocol: rtp-fec
      Description: "Primary RTP source"
      IngestPort: 5000
      WhitelistCidr: "192.168.1.0/24"
    sourceFailoverConfig:
      FailoverMode: FAILOVER
      State: ENABLED
      RecoveryWindow: 3000
      SourcePriority:
        primarySource: primary-source
    maintenance:
      MaintenanceDay: Sunday
      MaintenanceStartHour: "02:00"
```

### Flow with VPC Interface

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Flow
metadata:
  name: vpc-flow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    availabilityZone: us-east-1a
    source:
      Name: vpc-source
      Protocol: rtp
      Description: "Source from VPC"
      VpcInterfaceName: studio-interface
    vpcInterfaces:
      - Name: studio-interface
        RoleArn: arn:aws:iam::123456789012:role/MediaConnectVPCRole
        SecurityGroupIds:
          - sg-12345678
          - sg-87654321
        SubnetId: subnet-12345678
        NetworkInterfaceType: ena
    tags:
      Environment: production
      Network: vpc
```

### Flow with Media Streams

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Flow
metadata:
  name: multistream-flow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    source:
      Name: multistream-source
      Protocol: st2110-jpegxs
      Description: "ST 2110 source with multiple streams"
      IngestPort: 5000
    mediaStreams:
      - MediaStreamId: 1
        MediaStreamName: video-stream
        MediaStreamType: video
        Description: "Primary video stream"
        VideoFormat: "1080p"
        ClockRate: 90000
        Fmt: 96
      - MediaStreamId: 2
        MediaStreamName: audio-stream-left
        MediaStreamType: audio
        Description: "Left audio channel"
        ClockRate: 48000
        Fmt: 97
      - MediaStreamId: 3
        MediaStreamName: audio-stream-right
        MediaStreamType: audio
        Description: "Right audio channel"
        ClockRate: 48000
        Fmt: 98
```

### Flow with Gateway Bridge Source

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Flow
metadata:
  name: bridge-source-flow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    source:
      Name: bridge-source
      Description: "Source from MediaConnect bridge"
      GatewayBridgeSource:
        BridgeArn: arn:aws:mediaconnect:us-east-1:123456789012:bridge/example-bridge
        VpcInterfaceAttachment:
          VpcInterfaceName: bridge-vpc-interface
    vpcInterfaces:
      - Name: bridge-vpc-interface
        RoleArn: arn:aws:iam::123456789012:role/MediaConnectVPCRole
        SecurityGroupIds:
          - sg-bridge123
        SubnetId: subnet-bridge123
```

## Status

The flow status includes information about the AWS resource state and flow availability zone.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: Available
  atProvider:
    # AWS-specific resource information
    flowAvailabilityZone: us-east-1a
    properties: {}
```

## AWS Documentation

For more detailed information about MediaConnect flows, refer to the official AWS documentation:

- [AWS MediaConnect Flows](https://docs.aws.amazon.com/mediaconnect/latest/ug/flows.html)
- [AWS Cloud Control API - MediaConnect Flow](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations.html)
- [AWS::MediaConnect::Flow CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-mediaconnect-flow.html)

## Notes

- The `source` field is required and must contain at least a basic source configuration
- When using VPC interfaces, ensure the subnet is in the same availability zone as the flow
- Source failover requires careful configuration of source priorities
- Media streams are used for advanced ST 2110 and other professional broadcast protocols
- Encryption configuration requires proper IAM roles and Secrets Manager setup
- Flow size affects available features - `LARGE` enables NDI outputs
- Maintenance windows help ensure predictable service availability
