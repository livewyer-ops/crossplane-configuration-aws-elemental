# AWS Elemental Resources Documentation

This documentation covers the AWS Elemental resources available through the Crossplane configuration for AWS Elemental services. These resources enable you to provision and manage AWS Elemental media services infrastructure using Kubernetes-native declarative configuration.

## Overview

The AWS Elemental Crossplane configuration provides Composite Resource Definitions (XRDs) and Compositions for major AWS Elemental services, allowing you to create and manage live video workflows, transcoding pipelines, and content distribution systems using Kubernetes resources.

## Available Resources

### MediaConnect

AWS Elemental MediaConnect resources for live video transport and connectivity.

- **[Bridge](mediaconnect-bridge.md)** - Create bridges for connecting cloud and on-premises video workflows
- **[Flow](mediaconnect-flow.md)** - Transport live video content over IP networks with enterprise-grade security
- **[Gateway](mediaconnect-gateway.md)** - Connect on-premises equipment to MediaConnect flows in the cloud

### MediaConvert

AWS Elemental MediaConvert resources for video transcoding and processing.

- **[JobTemplate](mediaconvert-jobtemplate.md)** - Create job templates for standardizing video transcoding workflows

### MediaLive

AWS Elemental MediaLive resources for live video processing and encoding.

- **[MultiplexProgram](medialive-multiplexprogram.md)** - Create multiplex programs for combining multiple video streams
- **[Network](medialive-network.md)** - Manage IP address pools and routing configurations for live video workflows

### MediaPackage

AWS Elemental MediaPackage resources for video packaging and delivery.

- **[OriginEndpoint](mediapackage-originendpoint.md)** - Package and deliver live video content in various streaming formats

### MediaPackage V2

Next-generation AWS Elemental MediaPackage resources with enhanced features.

- **[Channel](mediapackagev2-channel.md)** - Ingest and process live video content with enhanced performance
- **[ChannelGroup](mediapackagev2-channelgroup.md)** - Organize and manage collections of related channels

### Workflow Orchestration

Advanced workflow orchestration capabilities for complex media pipelines.

- **[Event](event.md)** - Event-driven workflow execution with template-based configuration and parameterization
- **[WorkflowTemplate](workflowtemplate.md)** - Reusable workflow templates for common media processing patterns
- **[Workflow](workflow.md)** - Create complex, multi-step workflows with resource dependencies and dynamic value injection

### Future Resources (Planned)

- **MediaStore** - Storage service optimized for media workloads
- **MediaTailor** - Video personalization and monetization

## Resource Types

Each AWS Elemental resource is available in two forms:

### Composite Resources (Cluster-Scoped)

- **Prefix**: `X` (e.g., `XFlow`, `XBridge`, `XEvent`)
- **Scope**: Cluster-wide
- **Usage**: Direct creation of resources across the cluster
- **API Group**: `*.aws.livewyer.io`

### Claims (Namespace-Scoped)

- **Prefix**: None (e.g., `Flow`, `Bridge`, `Event`)
- **Scope**: Namespace-specific
- **Usage**: Application-specific resource claims within namespaces
- **API Group**: `*.aws.livewyer.io`

## Getting Started

### Prerequisites

1. **Crossplane Installation**: Ensure Crossplane is installed in your Kubernetes cluster
2. **AWS Provider**: Configure the AWS provider with appropriate credentials
3. **Provider Configuration**: Create ProviderConfig resources for AWS authentication

### Basic Usage

1. **Create a ProviderConfig**:

```yaml
apiVersion: aws.crossplane.io/v1beta1
kind: ProviderConfig
metadata:
  name: aws-provider-config
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: aws-secret
      key: creds
```

2. **Create Resources**: Use the documented resources to build your media workflows

3. **Monitor Status**: Check resource status using standard Kubernetes commands:

```bash
kubectl get flows
kubectl describe flow my-flow
kubectl get events
kubectl describe event my-event
```

## Workflow Patterns

### Event-Driven Workflows

Use Events to trigger workflow execution based on templates:

```mermaid
graph LR
    A[Event] --> B[WorkflowTemplate]
    B --> C[Resource Creation]
    C --> D[Dependency Resolution]
    D --> E[Workflow Execution]
```

### Template-Based Architecture

```mermaid
graph TD
    A[WorkflowTemplate] --> B[Event Trigger]
    B --> C[Parameter Injection]
    C --> D[Step Execution]
    D --> E[Resource Provisioning]
    E --> F[Status Reporting]
```

### Complex Media Workflows

```mermaid
graph TD
    A[MediaConnect Gateway] --> B[MediaConnect Bridge]
    B --> C[MediaConnect Flow]
    C --> D[MediaLive Channel]
    D --> E[MediaPackage Channel]
    E --> F[MediaPackage OriginEndpoint]
    F --> G[Content Distribution]

    H[Event] --> I[WorkflowTemplate]
    I --> A
```

## Common Patterns

### Basic Live Streaming Workflow

```mermaid
graph LR
    A[MediaConnect Flow] --> B[MediaPackage Channel]
    B --> C[MediaPackage OriginEndpoint]
    C --> D[CDN Distribution]
```

### Complex Broadcast Workflow

```mermaid
graph TD
    A[MediaConnect Gateway] --> B[MediaConnect Bridge]
    B --> C[MediaConnect Flow]
    C --> D[MediaLive Channel]
    D --> E[MediaPackage Channel]
    E --> F[MediaPackage OriginEndpoint]
    F --> G[Content Distribution]
```

### Event-Driven Live Streaming

```mermaid
graph TD
    A[Event Trigger] --> B[WorkflowTemplate Execution]
    B --> C[Network Setup]
    C --> D[MediaLive Configuration]
    D --> E[MediaPackage Setup]
    E --> F[Live Stream Ready]
```

## Resource Dependencies

Understanding resource dependencies is crucial for building successful workflows:

1. **MediaConnect Gateway** → **MediaConnect Bridge** (placement)
2. **MediaConnect Flow** → **MediaConnect Bridge** (source)
3. **MediaPackageV2 ChannelGroup** → **MediaPackageV2 Channel**
4. **WorkflowTemplate** → **Event** (template reference)
5. **Various Resources** → **Workflow** (orchestration)

## Best Practices

### Resource Naming

- Use descriptive, consistent naming conventions
- Include environment indicators (prod, dev, staging)
- Use organization-specific prefixes
- Avoid special characters that might cause issues

### Resource Organization

- Group related resources in the same namespace
- Use labels and annotations for resource organization
- Implement proper tagging strategies for cost tracking
- Document resource purposes and dependencies

### Workflow Design

- Use WorkflowTemplates for reusable patterns
- Leverage Events for trigger-based execution
- Implement proper parameter validation
- Design for idempotency and retry scenarios

### Security Considerations

- Use appropriate RBAC controls for resource access
- Implement network security policies
- Configure encryption where applicable
- Follow AWS security best practices
- Secure workflow template parameters

### Monitoring and Observability

- Implement comprehensive monitoring for all resources
- Set up alerting for resource failures
- Use proper logging and observability tools
- Monitor costs and resource utilization
- Track workflow execution and performance

## Workflow Orchestration Features

### Event-Driven Architecture

- **Trigger-based Execution**: Events trigger workflow execution based on external conditions
- **Template Parameterization**: WorkflowTemplates accept parameters for customization
- **Dynamic Configuration**: Runtime parameter injection into resource specifications

### Advanced Capabilities

- **Multi-Step Workflows**: Complex workflows with sequential and parallel execution
- **Dependency Management**: Automatic dependency resolution and value injection
- **Template Reusability**: Share common patterns across multiple events
- **Parameter Validation**: Built-in validation for workflow parameters

### Integration Patterns

- **GitOps Integration**: Version control workflow templates and events
- **CI/CD Pipeline Integration**: Trigger events from deployment pipelines
- **Monitoring Integration**: Automated workflow monitoring and alerting
- **Multi-Environment Support**: Deploy workflows across multiple environments

## Support and Troubleshooting

### Common Issues

1. **Resource Creation Failures**
   - Check ProviderConfig authentication
   - Verify AWS permissions
   - Review resource specifications for errors

2. **Dependency Resolution**
   - Ensure dependent resources exist and are ready
   - Check resource status conditions
   - Verify cross-references and ARNs

3. **Workflow Execution Issues**
   - Validate workflow template syntax
   - Check event parameter values
   - Review step execution order

4. **Performance Issues**
   - Monitor AWS service limits
   - Check resource configurations
   - Review network connectivity

### Getting Help

- Review individual resource documentation for detailed configuration options
- Check workflow template examples for common patterns
- Consult AWS service documentation for service-specific issues
- Use Kubernetes debugging commands for resource inspection

### Debugging Commands

```bash
# Check resource status
kubectl get events
kubectl describe event <event-name>
kubectl get workflowtemplates
kubectl describe workflowtemplate <template-name>

# Check workflow execution
kubectl get workflows
kubectl describe workflow <workflow-name>

# Monitor resource creation
kubectl get all -l elemental.aws.livewyer.io/workflow=<workflow-name>

# Check logs
kubectl logs -n crossplane-system deployment/crossplane
```

## Contributing

When contributing to this documentation:

1. Follow the established documentation structure
2. Include comprehensive examples for each use case
3. Document all configuration options and constraints
4. Provide troubleshooting guidance
5. Update this index when adding new resources
6. Test all examples and code snippets
7. Include workflow orchestration patterns where applicable

## API Versions

All resources use API version `v1alpha1` indicating they are in active development. Breaking changes may occur between versions. Always check the specific resource documentation for the most current configuration options and examples.

### Resource API Groups

- **MediaConnect**: `mediaconnect.aws.livewyer.io/v1alpha1`
- **MediaConvert**: `mediaconvert.aws.livewyer.io/v1alpha1`
- **MediaLive**: `medialive.aws.livewyer.io/v1alpha1`
- **MediaPackage**: `mediapackage.aws.livewyer.io/v1alpha1`
- **MediaPackage V2**: `mediapackagev2.aws.livewyer.io/v1alpha1`
- **Workflow Orchestration**: `elemental.aws.livewyer.io/v1alpha1`

## Examples Directory

The [`examples/`](../examples/) directory contains comprehensive examples:

- **Individual Resources**: Service-specific example files
- **Workflow Templates**: Reusable template examples
- **Events**: Event-driven workflow examples
- **Complex Workflows**: Multi-service integration examples
- **RBAC**: Role-based access control examples
- **Functions**: Composition function examples

---

_This documentation is maintained as part of the AWS Elemental Crossplane configuration. For the latest updates and additional resources, please refer to the individual resource documentation files and the main repository._
