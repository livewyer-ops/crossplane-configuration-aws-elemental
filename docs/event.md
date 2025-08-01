# Event

The Event resource allows you to create event-driven workflows that execute AWS Elemental media service configurations based on predefined WorkflowTemplates with dynamic parameterization.

## Overview

AWS Elemental Events provide a powerful way to trigger complex media workflows in response to external events or conditions. Events reference WorkflowTemplates and provide runtime parameters, enabling reusable, parameterized workflow execution across different environments and use cases. This approach supports event-driven architectures and simplifies the management of complex media processing pipelines.

## API Reference

### Resource Types

- **Composite Resource**: `XEvent` (cluster-scoped)
- **Claim**: `Event` (namespace-scoped)
- **API Version**: `elemental.aws.livewyer.io/v1alpha1`

### Specification

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `workflowTemplate` | object | Yes | WorkflowTemplate reference and additional configuration |
| `forProvider` | object | No | Global provider configuration and shared parameters |
| `deletionPolicy` | string | No | Deletion policy (`Delete`, `Orphan`) |
| `managementPolicies` | array | No | Management policies for the event |
| `providerConfigRef` | object | No | Provider configuration reference |
| `publishConnectionDetailsTo` | object | No | Connection details publishing configuration |
| `writeConnectionSecretsToNamespace` | string | No | Namespace for writing connection secrets |

### WorkflowTemplate Configuration

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | string | Yes | WorkflowTemplate identifier to execute |
| `parameters` | object | No | Runtime parameters for the WorkflowTemplate |

### ForProvider Configuration

Global configuration that can be shared across workflow resources:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `region` | string | No | AWS region for resource creation |
| `tags` | object | No | Default tags to apply to created resources |

## Examples

### Basic Event Execution

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: simple-live-stream
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
  workflowTemplate:
    id: basic-streaming-template
    parameters:
      streamName: "live-channel-1"
      bitrate: "5000000"
```

### Live Broadcasting Event

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: sports-broadcast-event
  namespace: broadcast
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    tags:
      Environment: production
      EventType: sports
  workflowTemplate:
    id: sports-broadcasting-template
    parameters:
      network: "192.168.10.0/24"
      iamRole: "arn:aws:iam::123456789012:role/MediaLiveAccessRole"
      eventName: "championship-game"
      primaryInput: "rtmp://encoder1.example.com/live"
      backupInput: "rtmp://encoder2.example.com/live"
      outputBitrates:
        - "8000000"
        - "5000000"
        - "2500000"
        - "1200000"
```

### Multi-Environment Event

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: cross-region-workflow
  namespace: default
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-west-2
    tags:
      Environment: production
      Scope: multi-region
  workflowTemplate:
    id: disaster-recovery-template
    parameters:
      primaryRegion: "us-west-2"
      backupRegion: "us-east-1"
      network: "10.0.0.0/16"
      iamRole: "arn:aws:iam::123456789012:role/MediaLiveAccessRole"
      channelName: "disaster-recovery-channel"
      replicationEnabled: true
```

### Development Event

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: dev-test-event
  namespace: development
spec:
  providerConfigRef:
    name: aws-dev-provider-config
  forProvider:
    region: us-west-2
    tags:
      Environment: development
      Purpose: testing
  workflowTemplate:
    id: development-workflow-template
    parameters:
      network: "192.168.100.0/24"
      iamRole: "arn:aws:iam::123456789012:role/MediaLiveDevRole"
      testMode: true
      lowCostMode: true
      streamDuration: "3600"  # 1 hour test
```

### Corporate Communication Event

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: company-townhall
  namespace: corporate
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    tags:
      Environment: production
      Department: communications
      Audience: internal
  workflowTemplate:
    id: corporate-streaming-template
    parameters:
      network: "172.16.0.0/16"
      iamRole: "arn:aws:iam::123456789012:role/MediaLiveCorporateRole"
      eventTitle: "Q3 Company Townhall"
      scheduledStart: "2024-01-15T14:00:00Z"
      estimatedDuration: "7200"  # 2 hours
      securityLevel: "high"
      recordingEnabled: true
      transcriptionEnabled: true
      audienceSize: "large"
```

### News Breaking Event

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: breaking-news-alert
  namespace: news
spec:
  providerConfigRef:
    name: aws-provider-config
  forProvider:
    region: us-east-1
    tags:
      Environment: production
      Priority: urgent
      ContentType: breaking-news
  workflowTemplate:
    id: breaking-news-template
    parameters:
      network: "10.1.0.0/24"
      iamRole: "arn:aws:iam::123456789012:role/MediaLiveNewsRole"
      urgencyLevel: "breaking"
      preemptScheduled: true
      multiLanguage: true
      socialMediaIntegration: true
      alertSystems:
        - "emergency-broadcast"
        - "mobile-push"
        - "web-banner"
```

## Status

The event status includes information about workflow execution and resource creation status.

```yaml
status:
  conditions:
    - type: Ready
      status: "True"
      reason: WorkflowExecuted
  atProvider:
    # Workflow execution details
    workflowStatus: "Completed"
    resourcesCreated: 5
    executionTime: "2024-01-15T10:30:00Z"
```

## Parameter Management

### Parameter Types

Events support various parameter types for WorkflowTemplate customization:

#### String Parameters
```yaml
parameters:
  streamName: "live-channel-1"
  description: "Live streaming channel"
  iamRole: "arn:aws:iam::123456789012:role/MediaLiveRole"
```

#### Numeric Parameters
```yaml
parameters:
  bitrate: 5000000
  segmentDuration: 6
  retryAttempts: 3
```

#### Boolean Parameters
```yaml
parameters:
  recordingEnabled: true
  encryptionEnabled: false
  debugMode: true
```

#### Array Parameters
```yaml
parameters:
  outputBitrates:
    - 8000000
    - 5000000
    - 2500000
  inputSources:
    - "rtmp://source1.example.com"
    - "rtmp://source2.example.com"
```

#### Object Parameters
```yaml
parameters:
  encryptionSettings:
    algorithm: "AES256"
    keyRotation: true
    provider: "SPEKE"
  networkConfig:
    cidr: "10.0.0.0/16"
    subnets: 2
    natGateway: true
```

### Parameter Validation

Parameters are validated against the WorkflowTemplate schema:

- **Required Parameters**: Must be provided in the event specification
- **Type Validation**: Parameters must match expected types
- **Value Constraints**: Parameters must meet any defined constraints
- **Default Values**: Missing optional parameters use template defaults

## Use Cases

### Live Event Broadcasting

Trigger live streaming workflows for scheduled events like sports, concerts, or conferences.

```yaml
workflowTemplate:
  id: live-event-template
  parameters:
    eventType: "sports"
    venue: "stadium-a"
    expectedViewers: 50000
```

### Breaking News Coverage

Rapidly deploy news streaming infrastructure for breaking news coverage.

```yaml
workflowTemplate:
  id: emergency-news-template
  parameters:
    urgency: "breaking"
    region: "affected-area"
    mobilization: "immediate"
```

### Corporate Communications

Deploy internal streaming for company meetings and training sessions.

```yaml
workflowTemplate:
  id: corporate-template
  parameters:
    audience: "internal"
    security: "enhanced"
    recording: "mandatory"
```

### Content Testing

Create test environments for media workflow validation and development.

```yaml
workflowTemplate:
  id: testing-template
  parameters:
    environment: "development"
    duration: "limited"
    monitoring: "detailed"
```

## Integration Patterns

### CI/CD Integration

```yaml
# Triggered from CI/CD pipeline
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: automated-deployment
  annotations:
    pipeline.id: "build-123"
    commit.sha: "abc123def456"
spec:
  workflowTemplate:
    id: deployment-template
    parameters:
      version: "v1.2.3"
      environment: "staging"
```

### External System Integration

```yaml
# Triggered by external monitoring system
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: failover-activation
  annotations:
    monitoring.alert: "primary-failure"
    automation.trigger: "disaster-recovery"
spec:
  workflowTemplate:
    id: failover-template
    parameters:
      triggerReason: "primary-system-failure"
      failoverRegion: "us-west-2"
```

### Scheduled Events

```yaml
# Triggered by cron job or scheduler
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: daily-news-broadcast
  annotations:
    schedule.cron: "0 18 * * *"  # 6 PM daily
spec:
  workflowTemplate:
    id: daily-news-template
    parameters:
      broadcastTime: "evening"
      duration: "1800"  # 30 minutes
```

## Best Practices

### Event Design

- **Descriptive Names**: Use clear, descriptive names for events
- **Proper Namespacing**: Organize events by purpose or environment
- **Parameter Validation**: Validate parameters before event creation
- **Documentation**: Document event purpose and parameter requirements

### Parameter Management

- **Sensitive Data**: Use Kubernetes secrets for sensitive parameters
- **Default Values**: Provide sensible defaults in WorkflowTemplates
- **Validation**: Implement parameter validation in templates
- **Documentation**: Document all available parameters

### Monitoring and Observability

- **Event Tracking**: Monitor event execution and success rates
- **Resource Monitoring**: Track resources created by events
- **Error Handling**: Implement proper error handling and alerting
- **Audit Trails**: Maintain audit logs for event execution

### Security Considerations

- **RBAC**: Implement proper role-based access controls
- **Parameter Security**: Secure sensitive parameters using secrets
- **Resource Isolation**: Use appropriate namespace isolation
- **Audit Logging**: Enable audit logging for event activities

## Troubleshooting

### Common Issues

1. **WorkflowTemplate Not Found**
   ```bash
   kubectl get workflowtemplates
   kubectl describe workflowtemplate <template-id>
   ```

2. **Parameter Validation Errors**
   ```bash
   kubectl describe event <event-name>
   kubectl logs -n crossplane-system deployment/crossplane
   ```

3. **Resource Creation Failures**
   ```bash
   kubectl get all -l elemental.aws.livewyer.io/event=<event-name>
   kubectl describe <resource-type> <resource-name>
   ```

4. **Permission Issues**
   ```bash
   kubectl auth can-i create events
   kubectl describe rolebinding -n <namespace>
   ```

### Debugging Commands

```bash
# Check event status
kubectl get events
kubectl describe event <event-name>

# Monitor workflow execution
kubectl get workflows -l elemental.aws.livewyer.io/event=<event-name>

# Check created resources
kubectl get all -l elemental.aws.livewyer.io/event=<event-name>

# Review logs
kubectl logs -n crossplane-system deployment/crossplane --tail=100
```

## AWS Documentation

For more information about the underlying AWS services used in workflows, refer to:

- [AWS MediaConnect Documentation](https://docs.aws.amazon.com/mediaconnect/)
- [AWS MediaLive Documentation](https://docs.aws.amazon.com/medialive/)
- [AWS MediaPackage Documentation](https://docs.aws.amazon.com/mediapackage/)
- [AWS MediaConvert Documentation](https://docs.aws.amazon.com/mediaconvert/)
- [Crossplane Documentation](https://docs.crossplane.io/)

## Notes

- **Template Dependency**: Events require existing WorkflowTemplates with matching IDs
- **Parameter Injection**: Parameters are injected into template resources at runtime
- **Idempotency**: Events should be designed for idempotent execution
- **Resource Cleanup**: Consider deletion policies for created resources
- **Monitoring**: Implement comprehensive monitoring for event-driven workflows
- **Version Control**: Store event definitions in version control systems
- **Testing**: Test events in development environments before production use
- **Documentation**: Maintain documentation for event parameters and usage patterns
