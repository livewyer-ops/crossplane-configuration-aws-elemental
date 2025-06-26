# MediaPackageV2 Channel

The MediaPackageV2 Channel resource allows you to create AWS Elemental MediaPackage V2 channels for ingesting and processing live video content with enhanced features and performance.

## Overview

AWS Elemental MediaPackage V2 channels provide the next generation of MediaPackage functionality with improved performance, enhanced features, and better integration with other AWS services. V2 channels support both HLS and CMAF input types and include advanced features like input switching based on media quality confidence scores (MQCS) and enhanced header configurations.

## API Reference

### Resource Types

- **Composite Resource**: `XChannel` (cluster-scoped)
- **Claim**: `Channel` (namespace-scoped)
- **API Version**: `mediapackagev2.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `channelGroupName` | string | Yes | Name of the channel group associated with the channel configuration (1-256 chars, alphanumeric, underscore, hyphen) |
| `description` | string | No | Description of the channel (0-1024 characters) |
| `inputType` | string | No | Input type for the channel (`HLS`, `CMAF`) - defaults to `HLS` |
| `inputSwitchConfiguration` | object | No | Configuration for input switching based on MQCS |
| `outputHeaderConfiguration` | object | No | Settings for common media server data (CMSD) headers in CDN responses |
| `region` | string | No | AWS region where the channel will be created |
| `tags` | object | No | Key-value pairs to associate with the channel |

### Input Switch Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `MQCSInputSwitching` | boolean | No | Enable input switching based on MQCS (default: true, valid only when InputType is CMAF) |

### Output Header Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `PublishMQCS` | boolean | No | Include MQCS in responses to CDN (valid only when InputType is CMAF) |

## Examples

### Basic HLS Channel

```yaml
apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
kind: Channel
metadata:
  name: basic-hls-channel
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelGroupName: my-channel-group
    description: "Basic HLS channel for live streaming"
    inputType: HLS
    tags:
      Environment: production
      Format: hls
```

### CMAF Channel with MQCS

```yaml
apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
kind: Channel
metadata:
  name: cmaf-mqcs-channel
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    channelGroupName: premium-channel-group
    description: "CMAF channel with Media Quality Confidence Score features"
    inputType: CMAF
    inputSwitchConfiguration:
      MQCSInputSwitching: true
    outputHeaderConfiguration:
      PublishMQCS: true
    tags:
      Environment: production
      Format: cmaf
      Features: mqcs-enabled
```

### Production Channel with Full Configuration

```yaml
apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
kind: Channel
metadata:
  name: production-channel
  namespace: broadcast
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelGroupName: broadcast-group
    description: "Production-ready channel for live broadcasting with quality monitoring"
    inputType: CMAF
    inputSwitchConfiguration:
      MQCSInputSwitching: true
    outputHeaderConfiguration:
      PublishMQCS: true
    tags:
      Environment: production
      Facility: broadcast-center
      SLA: premium
      MonitoringEnabled: "true"
```

### Development Channel

```yaml
apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
kind: Channel
metadata:
  name: dev-test-channel
  namespace: development
spec:
  providerConfigRef:
    name: aws-dev-provider-config
  forProvider:
    region: us-west-2
    channelGroupName: dev-channel-group
    description: "Development channel for testing workflows"
    inputType: HLS
    tags:
      Environment: development
      Purpose: testing
      CostOptimized: "true"
```

### Multi-Region Channel

```yaml
apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
kind: Channel
metadata:
  name: global-channel
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelGroupName: global-channel-group
    description: "Global channel for worldwide content distribution"
    inputType: CMAF
    inputSwitchConfiguration:
      MQCSInputSwitching: true
    outputHeaderConfiguration:
      PublishMQCS: true
    tags:
      Environment: production
      Scope: global
      Region: primary
      Redundancy: enabled
```

### Sports Broadcasting Channel

```yaml
apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
kind: Channel
metadata:
  name: sports-live-channel
  namespace: sports
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    channelGroupName: sports-content-group
    description: "Live sports broadcasting channel with adaptive quality switching"
    inputType: CMAF
    inputSwitchConfiguration:
      MQCSInputSwitching: true
    outputHeaderConfiguration:
      PublishMQCS: true
    tags:
      Environment: production
      Genre: sports
      Quality: adaptive
      Audience: live
```

## Status

The channel status includes information about the AWS resource state.

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

## Input Types

### HLS (HTTP Live Streaming)
- **Traditional format**: Widely supported across devices and platforms
- **Segment-based**: Uses individual segment files
- **Compatibility**: Excellent backward compatibility
- **Use cases**: General live streaming, mobile applications

### CMAF (Common Media Application Format)
- **Modern format**: Industry standard for efficient streaming
- **Single source**: Can generate both HLS and DASH from single ingest
- **Advanced features**: Supports MQCS and enhanced quality monitoring
- **Efficiency**: Reduced storage and bandwidth requirements

## Media Quality Confidence Score (MQCS)

### Overview
MQCS is a quality metric provided by AWS Elemental MediaLive that indicates the confidence level in the media quality of the input stream.

### Features
- **Input Switching**: Automatically switch between inputs based on quality confidence
- **CDN Integration**: Publish MQCS values to CDN for intelligent routing
- **Quality Monitoring**: Monitor stream quality in real-time
- **Adaptive Delivery**: Enable adaptive content delivery based on quality metrics

### Configuration
- **InputSwitchConfiguration**: Controls automatic input switching behavior
- **OutputHeaderConfiguration**: Controls whether MQCS is included in CDN responses
- **CMAF Requirement**: MQCS features are only available with CMAF input type

## Use Cases

### Live Event Broadcasting
Create channels for live events with automatic quality switching and monitoring capabilities.

### Sports Broadcasting
Utilize MQCS features for sports content where quality consistency is critical.

### News and Current Affairs
Implement reliable live news channels with backup input switching.

### Corporate Streaming
Deploy channels for corporate communications and training content.

### Multi-Platform Distribution
Use CMAF channels to efficiently serve content to multiple platforms and devices.

### Development and Testing
Create development channels for testing new workflows and configurations.

## Channel Groups Integration

MediaPackage V2 channels must be associated with a channel group, which provides:
- **Organizational structure** for managing related channels
- **Shared configuration** across channels in the group
- **Access control** and permissions management
- **Billing and cost tracking** at the group level

## Best Practices

### Performance Optimization
- Use CMAF input type for modern applications requiring advanced features
- Enable MQCS input switching for automatic quality management
- Configure appropriate CDN headers for optimal delivery

### Quality Management
- Monitor MQCS values to understand input quality trends
- Configure input switching thresholds based on your quality requirements
- Implement monitoring and alerting for channel health

### Cost Optimization
- Use HLS input type for simple use cases that don't require advanced features
- Properly size channels based on expected traffic
- Implement appropriate tagging for cost tracking

## AWS Documentation

For more detailed information about MediaPackage V2 channels, refer to the official AWS documentation:

- [AWS MediaPackage V2 Channels](https://docs.aws.amazon.com/mediapackage/latest/userguide/channels-v2.html)
- [AWS Cloud Control API - MediaPackageV2 Channel](https://docs.aws.amazon.com/cloudcontrolapi/latest/userguide/resource-operations.html)
- [AWS::MediaPackageV2::Channel CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-mediapackagev2-channel.html)

## Notes

- **Channel Group Dependency**: The channel group must exist before creating channels
- **Input Type Selection**: Choose input type based on your feature requirements and device compatibility
- **MQCS Features**: MQCS-related features are only available with CMAF input type
- **Regional Availability**: Ensure MediaPackage V2 is available in your target region
- **Naming Convention**: Channel group names must be alphanumeric with underscores and hyphens only
- **Description Limits**: Keep descriptions concise and within the 1024-character limit
- **Tag Strategy**: Use consistent tagging for resource management and cost tracking
- **Monitoring**: Implement comprehensive monitoring for production channels
