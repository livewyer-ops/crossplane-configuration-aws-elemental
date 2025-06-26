# MediaConvert JobTemplate

The MediaConvert JobTemplate resource allows you to create AWS Elemental MediaConvert job templates for standardizing video transcoding workflows.

## Overview

AWS Elemental MediaConvert job templates provide a way to standardize and reuse transcoding settings across multiple jobs. Templates contain predefined encoding settings, output configurations, and other parameters that can be applied to multiple transcoding jobs, ensuring consistency and simplifying workflow management.

## API Reference

### Resource Types

- **Composite Resource**: `XJobTemplate` (cluster-scoped)
- **Claim**: `JobTemplate` (namespace-scoped)
- **API Version**: `mediaconvert.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `outputGroups` | array | Yes | MediaConvert output group configurations |
| `category` | string | No | Optional category for organizing job templates |
| `description` | string | No | Optional description for the job template |
| `priority` | integer | No | Relative priority for jobs (-50 to 50) |
| `queue` | string | No | Queue for jobs created from this template |
| `region` | string | No | AWS region where the job template will be created |
| `tags` | object | No | Key-value pairs to associate with the job template |

### Output Groups Configuration

Output groups specify how MediaConvert processes and packages the output files.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Name` | string | No | Name of the output group |
| `OutputGroupSettings` | object | No | Settings for how the service handles outputs |
| `Outputs` | array | No | Individual output configurations within the group |

#### OutputGroupSettings

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Type` | string | No | Type of output group (`FILE_GROUP_SETTINGS`) |
| `FileGroupSettings` | object | No | Settings for file-based output groups |

#### Outputs Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `Extension` | string | No | File extension for outputs in this group |
| `NameModifier` | string | No | String added to end of each output filename |
| `Preset` | string | No | Preset for transcoding settings |

## Examples

### Basic Job Template

```yaml
apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
kind: JobTemplate
metadata:
  name: basic-transcoding-template
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    description: "Basic transcoding template for web delivery"
    category: "Web Delivery"
    priority: 0
    outputGroups:
      - Name: "WebDelivery"
        OutputGroupSettings:
          Type: "FILE_GROUP_SETTINGS"
          FileGroupSettings: {}
        Outputs:
          - Extension: "mp4"
            NameModifier: "_web"
            Preset: "System-Generic_Hd_Mp4_Av1_Aac_16x9_1920x1080p_24Hz_6Mbps"
    tags:
      Environment: production
      Purpose: web-delivery
```

### Multi-Output Job Template

```yaml
apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
kind: JobTemplate
metadata:
  name: multi-format-template
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    description: "Multi-format template for various delivery platforms"
    category: "Multi-Platform"
    priority: 10
    queue: "arn:aws:mediaconvert:us-west-2:123456789012:queues/Default"
    outputGroups:
      - Name: "HD_Outputs"
        OutputGroupSettings:
          Type: "FILE_GROUP_SETTINGS"
          FileGroupSettings: {}
        Outputs:
          - Extension: "mp4"
            NameModifier: "_1080p"
            Preset: "System-Generic_Hd_Mp4_Avc_Aac_16x9_1920x1080p_24Hz_6Mbps"
          - Extension: "mp4"
            NameModifier: "_720p"
            Preset: "System-Generic_Hd_Mp4_Avc_Aac_16x9_1280x720p_24Hz_3.5Mbps"
      - Name: "Mobile_Outputs"
        OutputGroupSettings:
          Type: "FILE_GROUP_SETTINGS"
          FileGroupSettings: {}
        Outputs:
          - Extension: "mp4"
            NameModifier: "_mobile"
            Preset: "System-Generic_Sd_Mp4_Avc_Aac_4x3_640x480p_24Hz_1.5Mbps"
    tags:
      Environment: production
      Platform: multi-device
      Quality: adaptive
```

### High-Priority Template

```yaml
apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
kind: JobTemplate
metadata:
  name: urgent-processing-template
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    description: "High-priority template for urgent content processing"
    category: "Urgent Processing"
    priority: 50
    queue: "arn:aws:mediaconvert:us-east-1:123456789012:queues/HighPriority"
    outputGroups:
      - Name: "UrgentDelivery"
        OutputGroupSettings:
          Type: "FILE_GROUP_SETTINGS"
          FileGroupSettings: {}
        Outputs:
          - Extension: "mp4"
            NameModifier: "_urgent"
            Preset: "System-Generic_Uhd_Mp4_Hevc_Aac_16x9_3840x2160p_24Hz_20Mbps"
    tags:
      Environment: production
      Priority: urgent
      SLA: express
```

### Broadcast Template

```yaml
apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
kind: JobTemplate
metadata:
  name: broadcast-template
  namespace: broadcast
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    description: "Template for broadcast-quality content"
    category: "Broadcast"
    priority: 25
    outputGroups:
      - Name: "BroadcastMasters"
        OutputGroupSettings:
          Type: "FILE_GROUP_SETTINGS"
          FileGroupSettings: {}
        Outputs:
          - Extension: "mxf"
            NameModifier: "_broadcast_master"
            Preset: "System-Broadcast_Mxf_Xdcam_Pcm_16x9_1920x1080i_50Hz_35Mbps"
          - Extension: "mov"
            NameModifier: "_proxy"
            Preset: "System-Generic_Hd_Mp4_Avc_Aac_16x9_1920x1080p_24Hz_6Mbps"
    tags:
      Environment: production
      Format: broadcast
      Quality: master
```

### Development Template

```yaml
apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
kind: JobTemplate
metadata:
  name: dev-test-template
  namespace: development
spec:
  providerConfigRef:
    name: aws-dev-provider-config
  forProvider:
    region: us-west-2
    description: "Development template for testing workflows"
    category: "Development"
    priority: -25
    outputGroups:
      - Name: "TestOutputs"
        OutputGroupSettings:
          Type: "FILE_GROUP_SETTINGS"
          FileGroupSettings: {}
        Outputs:
          - Extension: "mp4"
            NameModifier: "_test"
            Preset: "System-Generic_Sd_Mp4_Avc_Aac_4x3_640x480p_24Hz_1.5Mbps"
    tags:
      Environment: development
      Purpose: testing
      CostOptimized: "true"
```

## Status

The job template status includes information about the AWS resource state and CloudFormation stack details.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: Available
  atProvider:
    # AWS CloudFormation stack information
    outputs: {}
    stackStatus: CREATE_COMPLETE
```

## Priority System

MediaConvert job templates support a priority system to control job processing order:

- **Range**: -50 to 50
- **Default**: 0
- **Higher values**: Process first
- **Lower values**: Process after higher priority jobs
- **Use cases**:
  - `50`: Urgent, time-critical content
  - `25`: High priority content
  - `0`: Normal priority (default)
  - `-25`: Low priority, batch processing
  - `-50`: Background processing

## Common Presets

MediaConvert provides system presets for common use cases:

### Video Presets
- `System-Generic_Hd_Mp4_Avc_Aac_16x9_1920x1080p_24Hz_6Mbps`
- `System-Generic_Hd_Mp4_Avc_Aac_16x9_1280x720p_24Hz_3.5Mbps`
- `System-Generic_Uhd_Mp4_Hevc_Aac_16x9_3840x2160p_24Hz_20Mbps`
- `System-Generic_Sd_Mp4_Avc_Aac_4x3_640x480p_24Hz_1.5Mbps`

### Broadcast Presets
- `System-Broadcast_Mxf_Xdcam_Pcm_16x9_1920x1080i_50Hz_35Mbps`
- `System-Broadcast_Mxf_Mpeg2_Pcm_16x9_1920x1080i_25Hz_50Mbps`

### Streaming Presets
- `System-Ott_Hls_Ts_Avc_Aac_16x9_1920x1080p_30Hz_6.5Mbps`
- `System-Ott_Dash_Mp4_Avc_Aac_16x9_1920x1080p_30Hz_6.5Mbps`

## Use Cases

### Content Standardization
Ensure consistent encoding parameters across all transcoding jobs in your organization.

### Workflow Automation
Integrate with automated workflows to apply consistent processing to incoming content.

### Quality Control
Maintain consistent quality standards across different content types and delivery platforms.

### Cost Optimization
Use appropriate priority levels to optimize processing costs and resource utilization.

## AWS Documentation

For more detailed information about MediaConvert job templates, refer to the official AWS documentation:

- [AWS MediaConvert Job Templates](https://docs.aws.amazon.com/mediaconvert/latest/ug/working-with-job-templates.html)
- [AWS MediaConvert System Presets](https://docs.aws.amazon.com/mediaconvert/latest/ug/working-with-presets.html)
- [AWS::MediaConvert::JobTemplate CloudFormation Reference](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-mediaconvert-jobtemplate.html)

## Notes

- **Implementation**: This resource uses AWS CloudFormation Stack rather than Cloud Control API
- **Presets**: Use MediaConvert system presets or create custom presets beforehand
- **Queues**: Specify existing MediaConvert queues or use the default queue
- **Priority**: Higher priority jobs are processed before lower priority ones
- **Categories**: Use categories to organize templates for easier management
- **Output Groups**: Each output group can contain multiple outputs with different settings
- **File Extensions**: Specify appropriate file extensions for your target delivery platforms
- **Regional Availability**: Ensure MediaConvert is available in your target region
