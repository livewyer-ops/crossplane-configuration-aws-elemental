# WorkflowTemplate

The WorkflowTemplate resource allows you to create reusable workflow templates that define complex, multi-step AWS Elemental media service configurations with dependencies and parameterization for event-driven execution.

## Overview

AWS Elemental WorkflowTemplates provide a powerful way to define reusable, parameterized workflow patterns for media processing pipelines. Templates encapsulate complex multi-service configurations that can be executed by Events with different parameters, enabling standardization, reusability, and consistency across media workflows. WorkflowTemplates support Go templating for dynamic parameter injection and sophisticated dependency management between resources.

## API Reference

### Resource Types

- **Composite Resource**: `XWorkflowTemplate` (cluster-scoped)
- **Claim**: `WorkflowTemplate` (namespace-scoped)
- **API Version**: `elemental.aws.livewyer.io/v1alpha1`

### Specification

| Field                               | Type   | Required | Description                                         |
| ----------------------------------- | ------ | -------- | --------------------------------------------------- |
| `steps`                             | array  | Yes      | Array of workflow steps to execute sequentially     |
| `forProvider`                       | object | No       | Global provider configuration and shared parameters |
| `deletionPolicy`                    | string | No       | Deletion policy (`Delete`, `Orphan`)                |
| `managementPolicies`                | array  | No       | Management policies for the template                |
| `providerConfigRef`                 | object | No       | Provider configuration reference                    |
| `publishConnectionDetailsTo`        | object | No       | Connection details publishing configuration         |
| `writeConnectionSecretsToNamespace` | string | No       | Namespace for writing connection secrets            |

### Steps Configuration

Each step defines a logical grouping of resources to be created:

| Field       | Type   | Required | Description                                  |
| ----------- | ------ | -------- | -------------------------------------------- |
| `name`      | string | No       | Name of the workflow step for identification |
| `resources` | array  | Yes      | Array of resources to create in this step    |

### Resources Configuration

Each resource within a step defines a single AWS Elemental resource:

| Field       | Type   | Required | Description                                              |
| ----------- | ------ | -------- | -------------------------------------------------------- |
| `name`      | string | Yes      | Unique name of the resource within the workflow          |
| `spec`      | object | Yes      | Complete resource specification with Go template support |
| `dependsOn` | array  | No       | Array of dependencies on other resources in the workflow |

### Dependencies Configuration

Dependencies enable dynamic value injection between resources:

| Field    | Type   | Required | Description                                               |
| -------- | ------ | -------- | --------------------------------------------------------- |
| `key`    | string | Yes      | Target path where the dependency value should be injected |
| `type`   | string | Yes      | Data structure type for value injection                   |
| `source` | object | Yes      | Source resource and path for the dependency value         |

#### Source Configuration

Defines the source of dependency values:

| Field        | Type   | Required | Description                                     |
| ------------ | ------ | -------- | ----------------------------------------------- |
| `name`       | string | Yes      | Name of the source resource within the workflow |
| `apiVersion` | string | Yes      | API version of the source resource              |
| `kind`       | string | Yes      | Kind of the source resource                     |
| `key`        | string | Yes      | Path to the value in the source resource status |

## Examples

### Basic Streaming Template

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: basic-streaming-template
  namespace: default
spec:
  steps:
    - name: setup-network
      resources:
        - name: media-network
          spec:
            apiVersion: medialive.aws.livewyer.io/v1alpha1
            kind: Network
            spec:
              forProvider:
                ipPools:
                  - Cidr: "{{ network }}"
                tags:
                  Purpose: "{{ streamName }}"
    - name: create-packaging
      resources:
        - name: channel-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: "{{ streamName }}-group"
                description: "Channel group for {{ streamName }}"
        - name: package-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: Channel
            spec:
              forProvider:
                inputType: HLS
                description: "Channel for {{ streamName }}"
          dependsOn:
            - key: channelGroupName
              type: ""
              source:
                name: channel-group
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: ChannelGroup
                key: status.atProvider.properties.ChannelGroupName
```

### Advanced Media Workflow Template

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: advanced-media-workflow
  namespace: broadcast
spec:
  steps:
    - name: infrastructure-setup
      resources:
        - name: media-gateway
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Gateway
            spec:
              forProvider:
                egressCidrBlocks:
                  - "{{ network }}"
                networks:
                  - Name: "{{ networkName }}"
                    CidrBlock: "{{ network }}"
                tags:
                  Event: "{{ eventName }}"
                  Environment: "{{ environment }}"
        - name: medialive-network
          spec:
            apiVersion: medialive.aws.livewyer.io/v1alpha1
            kind: Network
            spec:
              forProvider:
                ipPools:
                  - Cidr: "{{ network }}"
                tags:
                  Purpose: "{{ eventName }}"
    - name: content-ingestion
      resources:
        - name: primary-flow
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: "{{ eventName }}-primary"
                  Protocol: "{{ inputProtocol }}"
                  IngestPort: "{{ primaryPort }}"
                  {{- if .encryptionEnabled }}
                  Decryption:
                    Algorithm: aes256
                    KeyType: static-key
                    SecretArn: "{{ encryptionSecret }}"
                    RoleArn: "{{ iamRole }}"
                  {{- end }}
                tags:
                  Type: primary
                  Event: "{{ eventName }}"
        {{- if .redundancyEnabled }}
        - name: backup-flow
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: "{{ eventName }}-backup"
                  Protocol: "{{ inputProtocol }}"
                  IngestPort: {{ backupPort }}
                tags:
                  Type: backup
                  Event: "{{ eventName }}"
        {{- end }}
    - name: content-processing
      resources:
        - name: transcoding-template
          spec:
            apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
            kind: JobTemplate
            spec:
              forProvider:
                description: "Transcoding template for {{ eventName }}"
                priority: {{ priority }}
                outputGroups:
                  - Name: "{{ eventName }}-outputs"
                    OutputGroupSettings:
                      Type: FILE_GROUP_SETTINGS
                    Outputs:
                      {{- range .outputBitrates }}
                      - Extension: mp4
                        NameModifier: "_{{  }}"
                        Preset: "System-Generic_Hd_Mp4_Avc_Aac_16x9_1920x1080p_24Hz_{{  }}bps"
                      {{- end }}
    - name: content-distribution
      resources:
        - name: package-channel-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: "{{ eventName }}-distribution"
                description: "Distribution group for {{ eventName }}"
        - name: distribution-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: Channel
            spec:
              forProvider:
                inputType: CMAF
                description: "Distribution channel for {{ eventName }}"
                inputSwitchConfiguration:
                  MQCSInputSwitching: {{ qualitySwitching }}
                outputHeaderConfiguration:
                  PublishMQCS: {{ publishMQCS }}
          dependsOn:
            - key: channelGroupName
              type: ""
              source:
                name: package-channel-group
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: ChannelGroup
                key: status.atProvider.properties.ChannelGroupName
        - name: content-bridge
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Bridge
            spec:
              forProvider:
                sources:
                  - FlowSource:
                      Name: "{{ eventName }}-bridge-source"
                egressGatewayBridge:
                  MaxBitrate: {{ maxBitrate }}
                tags:
                  Event: "{{ eventName }}"
          dependsOn:
            - key: sources.0.FlowSource.FlowArn
              type: ""
              source:
                name: primary-flow
                apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
                kind: Flow
                key: status.atProvider.properties.FlowArn
            - key: placementArn
              type: ""
              source:
                name: media-gateway
                apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
                kind: Gateway
                key: status.atProvider.properties.GatewayArn
```

### Sports Broadcasting Template

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: sports-broadcasting-template
  namespace: sports
spec:
  steps:
    - name: venue-setup
      resources:
        - name: venue-gateway
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Gateway
            spec:
              forProvider:
                egressCidrBlocks:
                  - "{{ venueNetwork }}"
                networks:
                  - Name: "{{ venue }}-network"
                    CidrBlock: "{{ venueNetwork }}"
                tags:
                  Venue: "{{ venue }}"
                  Sport: "{{ sport }}"
                  Event: "{{ eventName }}"
    - name: multi-camera-ingestion
      resources:
        {{- range $index, $camera := .cameras }}
        - name: "camera-{{ $index }}-flow"
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: "{{ camera.name }}"
                  Protocol: "{{ camera.protocol }}"
                  IngestPort: {{ $camera.port }}
                  {{- if $camera.encrypted }}
                  Decryption:
                    Algorithm: aes256
                    KeyType: static-key
                    SecretArn: "{{ encryptionSecret }}"
                    RoleArn: "{{ iamRole }}"
                  {{- end }}
                tags:
                  Camera: "{{ camera.name }}"
                  Position: "{{ camera.position }}"
                  Event: "{{ eventName }}"
        {{- end }}
    - name: broadcast-processing
      resources:
        - name: multiplex-program
          spec:
            apiVersion: medialive.aws.livewyer.io/v1alpha1
            kind: MultiplexProgram
            spec:
              forProvider:
                multiplexId: "{{ multiplexId }}"
                programName: "{{ eventName }}-program"
                multiplexProgramSettings:
                  ProgramNumber: {{ programNumber }}
                  ServiceDescriptor:
                    ProviderName: "{{ broadcaster }}"
                    ServiceName: "{{ eventName }}"
                  VideoSettings:
                    StatmuxSettings:
                      MaximumBitrate: {{ maxBitrate }}
                      MinimumBitrate: {{ minBitrate }}
                      Priority: {{ priority }}
    - name: distribution-setup
      resources:
        - name: broadcast-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: "{{ eventName }}-broadcast"
                description: "Broadcast distribution for {{ eventName }}"
        - name: live-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: Channel
            spec:
              forProvider:
                inputType: CMAF
                description: "Live channel for {{ eventName }}"
                inputSwitchConfiguration:
                  MQCSInputSwitching: true
                outputHeaderConfiguration:
                  PublishMQCS: true
          dependsOn:
            - key: channelGroupName
              type: ""
              source:
                name: broadcast-group
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: ChannelGroup
                key: status.atProvider.properties.ChannelGroupName
        - name: hls-endpoint
          spec:
            apiVersion: mediapackage.aws.livewyer.io/v1alpha1
            kind: OriginEndpoint
            spec:
              forProvider:
                description: "HLS endpoint for {{ eventName }}"
                hlsPackage:
                  AdMarkers: SCTE35_ENHANCED
                  PlaylistType: EVENT
                  PlaylistWindowSeconds: {{ playlistWindow }}
                  SegmentDurationSeconds: 6
                  ProgramDateTimeIntervalSeconds: 0
          dependsOn:
            - key: channelId
              type: ""
              source:
                name: live-channel
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: Channel
                key: status.atProvider.properties.ChannelName
```

### Corporate Communication Template

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: corporate-streaming-template
  namespace: corporate
spec:
  steps:
    - name: security-setup
      resources:
        - name: secure-gateway
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Gateway
            spec:
              forProvider:
                egressCidrBlocks:
                  - "{{ corporateNetwork }}"
                networks:
                  - Name: "corporate-secure"
                    CidrBlock: "{{ corporateNetwork }}"
                tags:
                  Security: enhanced
                  Department: "{{ department }}"
                  Event: "{{ eventTitle }}"
    - name: secure-ingestion
      resources:
        - name: executive-flow
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: "{{ eventTitle }}-source"
                  Protocol: srt-listener
                  IngestPort: 9001
                  MinLatency: 120
                  Decryption:
                    Algorithm: aes256
                    KeyType: static-key
                    SecretArn: "{{ encryptionSecret }}"
                    RoleArn: "{{ iamRole }}"
                tags:
                  Security: encrypted
                  Audience: internal
                  Event: "{{ eventTitle }}"
    - name: content-preparation
      resources:
        - name: corporate-template
          spec:
            apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
            kind: JobTemplate
            spec:
              forProvider:
                description: "Corporate template for {{ eventTitle }}"
                priority: {{ priority }}
                outputGroups:
                  - Name: CorporateOutputs
                    OutputGroupSettings:
                      Type: FILE_GROUP_SETTINGS
                    Outputs:
                      - Extension: mp4
                        NameModifier: "_corporate"
                        Preset: "System-Generic_Hd_Mp4_Avc_Aac_16x9_1920x1080p_24Hz_6Mbps"
                      {{- if .recordingEnabled }}
                      - Extension: mp4
                        NameModifier: "_archive"
                        Preset: "System-Generic_Hd_Mp4_Avc_Aac_16x9_1920x1080p_24Hz_3Mbps"
                      {{- end }}
    - name: internal-distribution
      resources:
        - name: internal-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: "corporate-internal"
                description: "Internal corporate communications"
        - name: secure-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: Channel
            spec:
              forProvider:
                inputType: HLS
                description: "Secure channel for {{ eventTitle }}"
          dependsOn:
            - key: channelGroupName
              type: ""
              source:
                name: internal-group
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: ChannelGroup
                key: status.atProvider.properties.ChannelGroupName
        {{- if .transcriptionEnabled }}
        - name: transcription-endpoint
          spec:
            apiVersion: mediapackage.aws.livewyer.io/v1alpha1
            kind: OriginEndpoint
            spec:
              forProvider:
                description: "Transcription endpoint for {{ eventTitle }}"
                hlsPackage:
                  AdMarkers: NONE
                  PlaylistType: EVENT
                  PlaylistWindowSeconds: 300
                  IncludeDvbSubtitles: true
          dependsOn:
            - key: channelId
              type: ""
              source:
                name: secure-channel
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: Channel
                key: status.atProvider.properties.ChannelName
        {{- end }}
```

### Development Testing Template

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: development-workflow-template
  namespace: development
spec:
  steps:
    - name: test-infrastructure
      resources:
        - name: dev-network
          spec:
            apiVersion: medialive.aws.livewyer.io/v1alpha1
            kind: Network
            spec:
              forProvider:
                ipPools:
                  - Cidr: "{{ network }}"
                tags:
                  Environment: development
                  Purpose: testing
                  CostOptimized: "true"
    - name: test-resources
      resources:
        - name: test-flow
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: test-source
                  Protocol: rtp
                  IngestPort: 5000
                tags:
                  Environment: development
                  TestMode: "{{ testMode }}"
        - name: test-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: dev-test-group
                description: "Development testing group"
        - name: test-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: Channel
            spec:
              forProvider:
                inputType: HLS
                description: "Test channel"
          dependsOn:
            - key: channelGroupName
              type: ""
              source:
                name: test-group
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: ChannelGroup
                key: status.atProvider.properties.ChannelGroupName
        {{- if .lowCostMode }}
        - name: basic-template
          spec:
            apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
            kind: JobTemplate
            spec:
              forProvider:
                description: "Basic testing template"
                priority: -25
                outputGroups:
                  - Name: TestOutputs
                    OutputGroupSettings:
                      Type: FILE_GROUP_SETTINGS
                    Outputs:
                      - Extension: mp4
                        NameModifier: "_test"
                        Preset: "System-Generic_Sd_Mp4_Avc_Aac_4x3_640x480p_24Hz_1.5Mbps"
        {{- end }}
```

## Status

The workflow template status includes information about template validation and readiness.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: TemplateValidated
  atProvider:
    # Template validation details
    stepsValidated: 4
    resourcesValidated: 12
    templateVersion: "v1.0.0"
```

## Go Template Features

### Template Syntax

WorkflowTemplates support Go template syntax for dynamic content:

#### Variable Interpolation

```yaml
name: "{{ resourceName }}"
description: "Resource for {{ eventName }}"
port: { { .portNumber } }
```

#### Conditional Logic

```yaml
{{- if .encryptionEnabled }}
encryption:
  algorithm: aes256
{{- end }}
```

#### Loops and Ranges

```yaml
{{- range .outputBitrates }}
- bitrate: {{  }}
{{- end }}

{{- range $index, $camera := .cameras }}
- name: "camera-{{ $index }}"
  position: "{{ camera.position }}"
{{- end }}
```

#### Complex Expressions

```yaml
priority: {{ if eq .environment "production" }}50{{ else }}0{{ end }}
maxBitrate: {{ mul .baseBitrate 2 }}
networkName: "{{ prefix }}-{{ environment }}-network"
```

### Template Functions

Available template functions include:

- **String Functions**: `upper`, `lower`, `title`, `trim`
- **Arithmetic Functions**: `add`, `sub`, `mul`, `div`
- **Comparison Functions**: `eq`, `ne`, `lt`, `gt`, `le`, `ge`
- **Logic Functions**: `and`, `or`, `not`
- **Type Functions**: `printf`, `print`, `println`

### Parameter Validation

Templates can include parameter validation:

```yaml
# Parameter requirements can be documented in annotations
metadata:
  annotations:
    parameters.required: "network,iamRole,eventName"
    parameters.optional: "encryptionEnabled,recordingEnabled"
    parameters.types: |
      network: string (CIDR format)
      iamRole: string (ARN format)
      eventName: string
      encryptionEnabled: boolean
      recordingEnabled: boolean
```

## Dependency Management

### Dependency Types

- **Value Injection**: Direct value copying from source to target
- **Path Resolution**: Deep object navigation using dot notation
- **Array Operations**: Adding or modifying array elements
- **Object Merging**: Merging object properties

### Dependency Patterns

#### Simple Value Injection

```yaml
dependsOn:
  - key: channelId
    type: ""
    source:
      name: source-channel
      apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
      kind: Channel
      key: status.atProvider.properties.ChannelName
```

#### Complex Path Resolution

```yaml
dependsOn:
  - key: sources.0.FlowSource.FlowArn
    type: ""
    source:
      name: primary-flow
      apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
      kind: Flow
      key: status.atProvider.properties.FlowArn
```

#### Array Element Addition

```yaml
dependsOn:
  - key: inputAttachments
    type: "array"
    source:
      name: media-input
      apiVersion: medialive.aws.upbound.io/v1beta2
      kind: Input
      key: status.atProvider.id
```

## Use Cases

### Live Event Templates

Create standardized templates for different types of live events (sports, concerts, conferences).

### Broadcast Workflow Templates

Define templates for traditional broadcast workflows with multiple cameras and distribution channels.

### Corporate Communication Templates

Build templates for internal corporate communications with enhanced security features.

### Testing and Development Templates

Create cost-optimized templates for development and testing environments.

### Disaster Recovery Templates

Design templates for automated disaster recovery and failover scenarios.

## Best Practices

### Template Design

- **Modularity**: Design templates with logical step groupings
- **Reusability**: Create flexible templates that work across environments
- **Parameter Validation**: Validate all required parameters
- **Documentation**: Document template purpose and parameters thoroughly

### Parameter Management

- **Naming Conventions**: Use consistent parameter naming
- **Default Values**: Provide sensible defaults where possible
- **Type Safety**: Ensure parameters match expected types
- **Validation**: Implement parameter validation logic

### Dependency Design

- **Clear Dependencies**: Make resource dependencies explicit
- **Path Accuracy**: Verify dependency paths are correct
- **Circular Avoidance**: Avoid circular dependency chains
- **Error Handling**: Handle dependency resolution failures gracefully

### Version Control

- **Template Versioning**: Version templates for backwards compatibility
- **Change Documentation**: Document template changes and migrations
- **Testing**: Test templates thoroughly before production use
- **Rollback Plans**: Maintain rollback procedures for template updates

## Integration Patterns

### CI/CD Integration

```yaml
# Template triggered by CI/CD pipeline
metadata:
  annotations:
    ci.pipeline: "media-workflow-pipeline"
    ci.stage: "deployment"
```

### GitOps Integration

```yaml
# Template stored in Git repository
metadata:
  annotations:
    gitops.repository: "https://github.com/company/media-templates"
    gitops.branch: "main"
    gitops.path: "templates/live-streaming"
```

### Environment Promotion

```yaml
# Template with environment-specific parameters
parameters:
  environment: "{{ targetEnvironment }}"
  region: '{{ if eq .targetEnvironment "production" }}us-east-1{{ else }}us-west-2{{ end }}'
```

## Troubleshooting

### Common Issues

1. **Template Validation Errors**

   ```bash
   kubectl describe workflowtemplate <template-name>
   kubectl logs -n crossplane-system deployment/crossplane
   ```

2. **Parameter Injection Failures**

   ```bash
   # Check template syntax
   kubectl get workflowtemplate <template-name> -o yaml
   ```

3. **Dependency Resolution Issues**

   ```bash
   # Verify resource dependencies
   kubectl get <resource-type> <resource-name> -o yaml
   ```

4. **Go Template Syntax Errors**
   ```bash
   # Validate template syntax locally
   kubectl apply --dry-run=client -f template.yaml
   ```

### Debugging Commands

```bash
# Check template status
kubectl get workflowtemplates
kubectl describe workflowtemplate <template-name>

# Validate template syntax
kubectl apply --dry-run=server -f template.yaml

# Monitor template usage
kubectl get events -l elemental.aws.livewyer.io/template=<template-name>

# Check related resources
kubectl get all -l elemental.aws.livewyer.io/template=<template-name>
```

## AWS Documentation

For more information about the underlying AWS services, refer to:

- [AWS MediaConnect Documentation](https://docs.aws.amazon.com/mediaconnect/)
- [AWS MediaLive Documentation](https://docs.aws.amazon.com/medialive/)
- [AWS MediaPackage Documentation](https://docs.aws.amazon.com/mediapackage/)
- [AWS MediaConvert Documentation](https://docs.aws.amazon.com/mediaconvert/)
- [Go Template Documentation](https://pkg.go.dev/text/template)
- [Crossplane Composition Documentation](https://docs.crossplane.io/latest/concepts/compositions/)

## Notes

- **Template Syntax**: Uses Go template syntax for dynamic parameter injection
- **Step Ordering**: Steps execute sequentially with dependency resolution within steps
- **Parameter Flexibility**: Supports complex parameter types including objects and arrays
- **Dependency Resolution**: Automatic dependency resolution between resources across steps
- **Validation**: Templates are validated for syntax and dependency correctness
- **Reusability**: Templates can be used by multiple events with different parameters
- **Version Control**: Store templates in version control for change management
- **Testing**: Test templates thoroughly in development environments
- **Documentation**: Maintain comprehensive documentation for template usage
