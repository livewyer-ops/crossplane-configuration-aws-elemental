# Workflow

The Workflow resource allows you to create complex, multi-step workflows that orchestrate the creation and configuration of multiple AWS Elemental resources with dependencies and dynamic value injection through direct resource specification.

## Overview

The Workflow resource provides a direct approach to defining and executing complex workflows involving multiple AWS Elemental resources. Unlike event-driven workflows (Events) or template-based workflows (WorkflowTemplates), Workflows contain complete resource specifications directly within the workflow definition. This approach supports dependency management, dynamic value injection between resources, and step-by-step execution of resource creation, enabling you to build sophisticated media processing pipelines with proper ordering and data flow between components.

## Workflow Types Comparison

This configuration provides three types of workflow orchestration:

- **Workflow** (this resource) - Direct workflow definition with complete resource specifications
- **[Event](event.md)** - Event-driven workflow execution using WorkflowTemplates with parameters
- **[WorkflowTemplate](workflowtemplate.md)** - Reusable workflow templates with Go templating and parameterization

Choose Workflows when you need direct control over resource specifications and don't require template reusability.

## API Reference

### Resource Types

- **Composite Resource**: `XWorkflow` (cluster-scoped)
- **Claim**: `Workflow` (namespace-scoped)
- **API Version**: `elemental.aws.livewyer.io/v1alpha1`

### Specification

| Field                               | Type   | Required | Description                                         |
| ----------------------------------- | ------ | -------- | --------------------------------------------------- |
| `steps`                             | array  | Yes      | Array of workflow steps to execute                  |
| `forProvider`                       | object | No       | Global provider configuration and shared parameters |
| `deletionPolicy`                    | string | No       | Deletion policy (`Delete`, `Orphan`)                |
| `managementPolicies`                | array  | No       | Management policies for the workflow                |
| `providerConfigRef`                 | object | No       | Provider configuration reference                    |
| `publishConnectionDetailsTo`        | object | No       | Connection details publishing configuration         |
| `writeConnectionSecretsToNamespace` | string | No       | Namespace for writing connection secrets            |

### Steps Configuration

Each step contains:

| Field       | Type   | Required | Description                               |
| ----------- | ------ | -------- | ----------------------------------------- |
| `name`      | string | No       | Name of the workflow step                 |
| `resources` | array  | Yes      | Array of resources to create in this step |

### Resources Configuration

Each resource contains:

| Field       | Type   | Required | Description                                      |
| ----------- | ------ | -------- | ------------------------------------------------ |
| `name`      | string | Yes      | Name of the resource                             |
| `spec`      | object | Yes      | Resource specification (varies by resource type) |
| `dependsOn` | array  | No       | Array of dependencies on other resources         |

### Dependencies Configuration

Each dependency contains:

| Field               | Type   | Required | Description                                               |
| ------------------- | ------ | -------- | --------------------------------------------------------- |
| `key`               | string | Yes      | Target path where the dependency value should be injected |
| `type`              | string | Yes      | Data structure type for value injection                   |
| `source`            | object | Yes      | Source resource and path for the dependency value         |
| `subKey`            | string | No       | Sub-key for list/object operations                        |
| `subKeyPatchMethod` | string | No       | Method for sub-key patching (`merge`)                     |

#### Source Configuration

| Field        | Type   | Required | Description                              |
| ------------ | ------ | -------- | ---------------------------------------- |
| `name`       | string | Yes      | Name of the source resource              |
| `apiVersion` | string | Yes      | API version of the source resource       |
| `kind`       | string | Yes      | Kind of the source resource              |
| `key`        | string | Yes      | Path to the value in the source resource |

## Examples

### Basic MediaConnect Workflow

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Workflow
metadata:
  name: basic-mediaconnect-workflow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
  steps:
    - name: create-gateway
      resources:
        - name: studio-gateway
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Gateway
            spec:
              forProvider:
                egressCidrBlocks:
                  - "10.0.0.0/16"
                networks:
                  - Name: studio-network
                    CidrBlock: "10.0.100.0/24"
                tags:
                  Purpose: workflow-demo
    - name: create-flow
      resources:
        - name: studio-flow
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: studio-source
                  Protocol: zixi-push
                  IngestPort: 2088
                tags:
                  Purpose: workflow-demo
```

### Complex Workflow with Dependencies

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Workflow
metadata:
  name: complex-mediaconnect-workflow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
  steps:
    - name: infrastructure
      resources:
        - name: main-gateway
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Gateway
            spec:
              forProvider:
                egressCidrBlocks:
                  - "10.0.0.0/8"
                networks:
                  - Name: primary-network
                    CidrBlock: "10.1.0.0/16"
                  - Name: backup-network
                    CidrBlock: "10.2.0.0/16"
                tags:
                  Environment: production
        - name: channel-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: production-group
                description: "Production channel group"
    - name: content-processing
      resources:
        - name: primary-flow
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: primary-source
                  Protocol: rtp-fec
                  IngestPort: 5000
                tags:
                  Type: primary
        - name: packaging-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: Channel
            spec:
              forProvider:
                inputType: CMAF
                description: "Channel for packaged content"
          dependsOn:
            - key: channelGroupName
              type: ""
              source:
                name: channel-group
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: ChannelGroup
                key: status.atProvider.properties.ChannelGroupName
    - name: distribution
      resources:
        - name: content-bridge
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Bridge
            spec:
              forProvider:
                sources:
                  - FlowSource:
                      Name: primary-bridge-source
                egressGatewayBridge:
                  MaxBitrate: 50000000
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
                name: main-gateway
                apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
                kind: Gateway
                key: status.atProvider.properties.GatewayArn
```

### Multi-Step Broadcast Workflow

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Workflow
metadata:
  name: broadcast-workflow
  namespace: broadcast
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
  steps:
    - name: setup-infrastructure
      resources:
        - name: broadcast-gateway
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Gateway
            spec:
              forProvider:
                egressCidrBlocks:
                  - "172.16.0.0/12"
                networks:
                  - Name: broadcast-network
                    CidrBlock: "172.16.1.0/24"
                tags:
                  Facility: broadcast-center
        - name: media-network
          spec:
            apiVersion: medialive.aws.livewyer.io/v1alpha1
            kind: Network
            spec:
              forProvider:
                ipPools:
                  - Cidr: "192.168.10.0/24"
                tags:
                  Purpose: media-processing
    - name: content-ingestion
      resources:
        - name: camera-feed-1
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: camera-1
                  Protocol: srt-listener
                  IngestPort: 9001
                tags:
                  Camera: camera-1
        - name: camera-feed-2
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Flow
            spec:
              forProvider:
                source:
                  Name: camera-2
                  Protocol: srt-listener
                  IngestPort: 9002
                tags:
                  Camera: camera-2
    - name: content-packaging
      resources:
        - name: broadcast-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: broadcast-content
                description: "Broadcast content packaging"
        - name: primary-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: Channel
            spec:
              forProvider:
                inputType: CMAF
                description: "Primary broadcast channel"
          dependsOn:
            - key: channelGroupName
              type: ""
              source:
                name: broadcast-group
                apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
                kind: ChannelGroup
                key: status.atProvider.properties.ChannelGroupName
    - name: transcoding-templates
      resources:
        - name: broadcast-template
          spec:
            apiVersion: mediaconvert.aws.livewyer.io/v1alpha1
            kind: JobTemplate
            spec:
              forProvider:
                description: "Broadcast quality transcoding"
                priority: 25
                outputGroups:
                  - Name: BroadcastOutputs
                    OutputGroupSettings:
                      Type: FILE_GROUP_SETTINGS
                    Outputs:
                      - Extension: mp4
                        NameModifier: _broadcast
                        Preset: System-Generic_Hd_Mp4_Avc_Aac_16x9_1920x1080p_24Hz_6Mbps
```

### Development and Testing Workflow

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Workflow
metadata:
  name: dev-test-workflow
  namespace: development
spec:
  providerConfigRef:
    name: aws-dev-provider-config
  forProvider:
    region: us-west-2
  steps:
    - name: dev-setup
      resources:
        - name: dev-gateway
          spec:
            apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
            kind: Gateway
            spec:
              forProvider:
                egressCidrBlocks:
                  - "192.168.0.0/16"
                networks:
                  - Name: dev-network
                    CidrBlock: "192.168.100.0/24"
                tags:
                  Environment: development
        - name: test-group
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: dev-test-group
                description: "Development testing group"
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
```

## Status

The workflow status includes information about all composed resources and their states.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: Available
  # Individual resource statuses are available through the Crossplane resource status
```

## Dependency Management

### Dependency Types

The workflow supports various dependency types for value injection:

- **String values**: Direct value injection
- **List operations**: Adding items to arrays
- **Object operations**: Merging or setting object properties
- **Nested paths**: Deep object/array path resolution

### Dependency Resolution

Dependencies are resolved using dot notation paths:

- `status.atProvider.properties.FlowArn` - Navigate to nested properties
- `metadata.name` - Access resource metadata
- `spec.forProvider.region` - Access specification values

### Sub-Key Operations

For complex dependency scenarios:

- **subKey**: Specify sub-keys for list/object operations
- **subKeyPatchMethod**: Control how sub-keys are merged (`merge` for object merging)

## Use Cases

### Direct Resource Orchestration

Create workflows where you need complete control over individual resource specifications without templating.

### One-Off Deployments

Orchestrate specific deployments that don't require reusability across multiple environments.

### Complex Dependency Chains

Implement workflows with intricate resource dependencies that benefit from explicit specification.

### Legacy Workflow Migration

Migrate existing infrastructure-as-code workflows to Crossplane without template abstraction.

### Development and Prototyping

Develop and prototype new workflow patterns before converting to reusable templates.

### Environment-Specific Workflows

Create workflows tailored to specific environments where template reusability isn't needed.

## Best Practices

### When to Use Workflows vs Events/Templates

**Use Workflows when:**

- You need direct control over resource specifications
- The workflow is environment-specific and won't be reused
- You're prototyping or developing new patterns
- You have complex, one-off dependency requirements

**Use Events + WorkflowTemplates when:**

- You need reusable workflow patterns
- You want parameterized, template-based execution
- You're implementing event-driven architectures
- You need consistent workflows across environments

### Workflow Design

- **Logical Steps**: Organize resources into logical steps
- **Clear Dependencies**: Explicitly define resource dependencies
- **Error Handling**: Consider failure scenarios and recovery
- **Documentation**: Document workflow purpose and dependencies
- **Consider Templates**: Evaluate if this workflow could benefit from being converted to a WorkflowTemplate

### Resource Naming

- **Consistent Naming**: Use consistent naming conventions
- **Descriptive Names**: Use descriptive resource names
- **Environment Prefixes**: Include environment indicators
- **Version Control**: Track workflow changes

### Dependency Management

- **Explicit Dependencies**: Always specify required dependencies
- **Path Validation**: Validate dependency paths
- **Circular Dependencies**: Avoid circular dependency chains
- **Testing**: Test dependency resolution thoroughly

## Troubleshooting

### Common Issues

1. **Dependency Resolution Failures**
   - Verify source resource exists and is ready
   - Check dependency path syntax
   - Ensure source resource has required fields

2. **Resource Creation Order**
   - Review step organization
   - Verify dependencies are in previous steps
   - Check for circular dependencies

3. **Value Injection Problems**
   - Validate target path exists
   - Check data type compatibility
   - Review sub-key operations

## Related Resources

### Other Workflow Types

- **[Event](event.md)** - Event-driven workflow execution with template-based configuration
- **[WorkflowTemplate](workflowtemplate.md)** - Reusable workflow templates with Go templating and parameterization

### AWS Documentation

For more information about the underlying AWS resources used in workflows, refer to:

- [AWS MediaConnect Documentation](https://docs.aws.amazon.com/mediaconnect/)
- [AWS MediaPackage Documentation](https://docs.aws.amazon.com/mediapackage/)
- [AWS MediaLive Documentation](https://docs.aws.amazon.com/medialive/)
- [AWS MediaConvert Documentation](https://docs.aws.amazon.com/mediaconvert/)
- [Crossplane Composition Documentation](https://docs.crossplane.io/latest/concepts/compositions/)

## Notes

- **Direct Definition**: Contains complete resource specifications without templating abstraction
- **Resource Ordering**: Resources are created in step order with proper dependency resolution
- **Global Configuration**: Use `forProvider` for shared configuration across resources
- **Flexible Dependencies**: Support complex dependency patterns with dynamic value injection
- **No Templating**: Unlike WorkflowTemplates, uses direct resource specifications
- **Single-Use Focus**: Best suited for specific, non-reusable workflow patterns
- **Namespace Scoped**: Workflows are namespace-scoped for multi-tenant environments
- **Provider Integration**: Seamlessly integrates with Crossplane provider ecosystem
- **Monitoring**: Monitor workflow execution through Crossplane resource status
- **Template Migration**: Consider migrating to WorkflowTemplates for reusable patterns
- **Rollback**: Consider rollback strategies for failed workflow executions
