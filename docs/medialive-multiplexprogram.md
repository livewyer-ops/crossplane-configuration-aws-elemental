# MediaLive MultiplexProgram

The MediaLive MultiplexProgram resource allows you to create AWS Elemental MediaLive multiplex programs for combining multiple video streams into a single transport stream.

## Overview

AWS Elemental MediaLive multiplex programs define how individual channels are combined into a multiplex output. A multiplex can contain multiple programs, each representing a different video service or channel. Programs specify encoding settings, packet identifiers, and other transport stream parameters.

## API Reference

### Resource Types

- **Composite Resource**: `XMultiplexProgram` (cluster-scoped)
- **Claim**: `MultiplexProgram` (namespace-scoped)
- **API Version**: `medialive.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `multiplexId` | string | Yes | The unique ID of the multiplex |
| `programName` | string | Yes | The name of the multiplex program |
| `multiplexProgramSettings` | object | No | Multiplex program settings configuration |
| `packetIdentifiersMap` | object | No | Packet identifiers map for the program |
| `pipelineDetails` | array | No | Current source for pipelines in the multiplex |
| `preferredChannelPipeline` | string | No | Preferred pipeline for program ingest |
| `region` | string | No | AWS region (defaults to us-east-1) |
| `tags` | object | No | Key-value pairs to associate with the program |

### Multiplex Program Settings

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ProgramNumber` | integer | Yes | Unique program number (0-65535) |
| `PreferredChannelPipeline` | string | No | Preferred pipeline (`CURRENTLY_ACTIVE`, `PIPELINE_0`, `PIPELINE_1`) |
| `ServiceDescriptor` | object | No | Transport stream service descriptor configuration |
| `VideoSettings` | object | No | Program video settings configuration |

#### Service Descriptor Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ProviderName` | string | No | Name of the provider (1-256 characters) |
| `ServiceName` | string | No | Name of the service (1-256 characters) |

#### Video Settings Configuration

Video settings support either constant bitrate or statmux (statistical multiplexing):

**Constant Bitrate:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `ConstantBitrate` | integer | Yes | Constant bitrate for video encode (100000-100000000 bps) |

**Statmux Settings:**
| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `StatmuxSettings.MaximumBitrate` | integer | No | Maximum statmux bitrate (100000-100000000 bps) |
| `StatmuxSettings.MinimumBitrate` | integer | No | Minimum statmux bitrate (100000-100000000 bps) |
| `StatmuxSettings.Priority` | integer | No | Priority value for video stream (-5 to 5) |

### Packet Identifiers Map

| Field | Type | Description |
|-------|------|-------------|
| `AudioPids` | array | Audio packet identifiers (32-8190) |
| `DvbSubPids` | array | DVB subtitle packet identifiers |
| `DvbTeletextPid` | integer | DVB teletext packet identifier |
| `EtvPlatformPid` | integer | ETV platform packet identifier |
| `EtvSignalPid` | integer | ETV signal packet identifier |
| `KlvDataPids` | array | KLV data packet identifiers |
| `PcrPid` | integer | PCR packet identifier |
| `PmtPid` | integer | PMT packet identifier |
| `PrivateMetadataPid` | integer | Private metadata packet identifier |
| `Scte27Pids` | array | SCTE-27 packet identifiers |
| `Scte35Pid` | integer | SCTE-35 packet identifier |
| `TimedMetadataPid` | integer | Timed metadata packet identifier |
| `VideoPid` | integer | Video packet identifier |

## Examples

### Basic Multiplex Program

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: MultiplexProgram
metadata:
  name: basic-program
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    multiplexId: "12345678"
    programName: "Channel1"
    multiplexProgramSettings:
      ProgramNumber: 1
      ServiceDescriptor:
        ProviderName: "MyProvider"
        ServiceName: "Channel 1 HD"
      VideoSettings:
        ConstantBitrate: 5000000
    tags:
      Environment: production
      Channel: channel-1
```

### Program with Statmux Video Settings

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: MultiplexProgram
metadata:
  name: statmux-program
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    multiplexId: "87654321"
    programName: "SportsChannel"
    multiplexProgramSettings:
      ProgramNumber: 2
      PreferredChannelPipeline: "PIPELINE_0"
      ServiceDescriptor:
        ProviderName: "Sports Network"
        ServiceName: "Sports HD"
      VideoSettings:
        StatmuxSettings:
          MaximumBitrate: 15000000
          MinimumBitrate: 3000000
          Priority: 2
    tags:
      Environment: production
      Genre: sports
      Quality: hd
```

### Program with Packet Identifiers

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: MultiplexProgram
metadata:
  name: detailed-program
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    multiplexId: "11223344"
    programName: "NewsChannel"
    multiplexProgramSettings:
      ProgramNumber: 3
      ServiceDescriptor:
        ProviderName: "News Corp"
        ServiceName: "24/7 News"
      VideoSettings:
        ConstantBitrate: 8000000
    packetIdentifiersMap:
      VideoPid: 256
      AudioPids:
        - 257
        - 258
      PcrPid: 256
      PmtPid: 100
      Scte35Pid: 300
      DvbSubPids:
        - 400
        - 401
    preferredChannelPipeline: "PIPELINE_1"
    tags:
      Environment: production
      Content: news
      Language: multi
```

### Program with Pipeline Details

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: MultiplexProgram
metadata:
  name: pipeline-program
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    multiplexId: "55667788"
    programName: "EntertainmentChannel"
    multiplexProgramSettings:
      ProgramNumber: 4
      ServiceDescriptor:
        ProviderName: "Entertainment Ltd"
        ServiceName: "Prime Entertainment"
      VideoSettings:
        StatmuxSettings:
          MaximumBitrate: 12000000
          MinimumBitrate: 4000000
          Priority: 0
    pipelineDetails:
      - PipelineId: "pipeline-0"
        ActiveChannelPipeline: "PIPELINE_0"
      - PipelineId: "pipeline-1"
        ActiveChannelPipeline: "PIPELINE_1"
    preferredChannelPipeline: "CURRENTLY_ACTIVE"
    tags:
      Environment: production
      Genre: entertainment
      Redundancy: enabled
```

### Development Program

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: MultiplexProgram
metadata:
  name: dev-test-program
  namespace: development
spec:
  providerConfigRef:
    name: aws-dev-provider-config
  forProvider:
    region: us-west-2
    multiplexId: "99887766"
    programName: "TestChannel"
    multiplexProgramSettings:
      ProgramNumber: 999
      ServiceDescriptor:
        ProviderName: "Dev Team"
        ServiceName: "Test Service"
      VideoSettings:
        ConstantBitrate: 2000000
    packetIdentifiersMap:
      VideoPid: 500
      AudioPids:
        - 501
      PcrPid: 500
      PmtPid: 200
    tags:
      Environment: development
      Purpose: testing
      Owner: dev-team
```

## Status

The multiplex program status includes information about the AWS resource state.

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

## Pipeline Management

### Preferred Channel Pipeline Options

- `CURRENTLY_ACTIVE`: Use whichever pipeline is currently active
- `PIPELINE_0`: Prefer pipeline 0 for ingest
- `PIPELINE_1`: Prefer pipeline 1 for ingest

When a preferred pipeline becomes unhealthy, the multiplex automatically switches to the other pipeline and will switch back when the preferred pipeline becomes healthy again.

## Packet Identifier (PID) Management

### Common PID Ranges
- **Video PIDs**: Typically 256-511
- **Audio PIDs**: Typically 512-767
- **Data PIDs**: Typically 768-1023
- **Reserved PIDs**: 0-31 (avoid using)

### Best Practices
- Keep PIDs consistent across similar programs
- Use sequential PIDs for related streams (e.g., stereo audio pairs)
- Document PID assignments for troubleshooting
- Avoid conflicts with standard reserved PIDs

## Use Cases

### Broadcast Television
Combine multiple television channels into a single transport stream for cable/satellite distribution.

### IPTV Services
Package multiple video services for IP-based delivery to set-top boxes.

### Digital Signage
Distribute multiple content streams to digital signage networks.

### Sports Broadcasting
Manage multiple camera feeds and audio tracks for live sports events.

## AWS Documentation

For more detailed information about MediaLive multiplex programs, refer to the official AWS documentation:

- [AWS MediaLive Multiplexes](https://docs.aws.amazon.com/medialive/latest/ug/multiplex.html)
- [AWS Cloud Control API - MediaLive MultiplexProgram](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations.html)
- [AWS::MediaLive::Multiplexprogram CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-medialive-multiplexprogram.html)

## Notes

- **Multiplex Dependency**: The multiplex must exist before creating programs
- **Program Numbers**: Must be unique within the multiplex (0-65535)
- **Video Settings**: Choose either constant bitrate or statmux, not both
- **PID Management**: Ensure PIDs don't conflict within the multiplex
- **Pipeline Redundancy**: Configure pipeline preferences for high availability
- **Service Descriptors**: Help viewers identify channels in electronic program guides
- **Regional Deployment**: Default region is us-east-1 if not specified
- **Statmux Benefits**: Statistical multiplexing optimizes bandwidth usage across programs
