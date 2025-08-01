# Crossplane Configuration for AWS Elemental

A comprehensive Crossplane Configuration package that provides Composite Resource Definitions (XRDs) for AWS Elemental media services on Kubernetes.

## Overview

This configuration package enables you to provision and manage AWS Elemental media services infrastructure using Kubernetes-native APIs through Crossplane. It provides high-level abstractions for complex media workflows while maintaining the flexibility and power of AWS Elemental services.

## Supported AWS Elemental Services

- **AWS Elemental MediaConnect** - Secure, reliable live video transport
- **AWS Elemental MediaConvert** - File-based video transcoding service
- **AWS Elemental MediaLive** - Live video processing service
- **AWS Elemental MediaPackage** - Video origination and packaging service
- **AWS Elemental MediaPackage V2** - Next-generation video packaging
- **AWS Elemental MediaStore** - Storage service optimized for media (planned)
- **AWS Elemental MediaTailor** - Video personalization and monetization (planned)

## Features

- **Event-Driven Workflows**: Define event-driven media workflows with template-based orchestration
- **Workflow Templates**: Reusable workflow templates for common media processing patterns
- **Multi-Service Integration**: Seamless integration between different AWS Elemental services
- **Kubernetes Native**: Manage media infrastructure using familiar Kubernetes tooling
- **GitOps Ready**: Version control and automate media infrastructure deployments
- **Composition Functions**: Advanced resource composition using Go templating and auto-ready functions

## Available Resources

### MediaConnect Resources

- **Bridge** - Connect cloud and on-premises video workflows
- **Flow** - Transport live video content over IP networks
- **Gateway** - Connect on-premises equipment to MediaConnect flows

### MediaConvert Resources

- **JobTemplate** - Standardize video transcoding workflows

### MediaLive Resources

- **MultiplexProgram** - Combine multiple video streams into transport streams
- **Network** - Manage IP address pools and routing configurations

### MediaPackage Resources

- **OriginEndpoint** - Package and deliver live video content

### MediaPackage V2 Resources

- **Channel** - Ingest and process live video content with enhanced features
- **ChannelGroup** - Organize and manage collections of related channels

### Workflow Orchestration

- **Event** - Event-driven workflow execution with template-based configuration
- **WorkflowTemplate** - Reusable workflow templates with parameterization
- **Workflow** - Complex multi-step workflows with resource dependencies

## Prerequisites

Before using this configuration, ensure you have:

1. **Kubernetes Cluster** (v1.20+)
2. **Crossplane** (v1.20.0+) installed in your cluster
3. **AWS Account** with appropriate permissions for Elemental services
4. **AWS CLI** configured (for initial setup)

## Installation

### 1. Install Crossplane

If you haven't already installed Crossplane:

```bash
helm repo add crossplane-stable https://charts.crossplane.io/stable
helm repo update
helm install crossplane crossplane-stable/crossplane \
  --namespace crossplane-system \
  --create-namespace
```

### 2. Install Required Providers and Functions

The configuration will automatically install dependencies, but you can install them manually:

```bash
# Apply the functions
kubectl apply -f examples/functions.yaml
# Apply the providers
kubectl apply -f examples/providers.yaml
```

### 3. Install the Configuration

```bash
kubectl apply -f - <<EOF
apiVersion: pkg.crossplane.io/v1
kind: Configuration
metadata:
  name: configuration-aws-elemental
spec:
  package: xpkg.upbound.io/livewyer-ops/configuration-aws-elemental:latest
EOF
```

### 4. Apply RBAC Permissions

```bash
kubectl apply -f examples/rbac.yaml
```

## Environment Setup

### Kubernetes ProviderConfig Setup

#### 1. If provider kubernetes running in the cluster

```bash
SA=$(kubectl -n crossplane-system get sa -o name | grep provider-kubernetes | sed -e 's|serviceaccount\/|crossplane-system:|g')
kubectl create clusterrolebinding provider-kubernetes-admin-binding --clusterrole cluster-admin --serviceaccount="${SA}"
```

#### 2. Apply ProviderConfig

```bash
kubectl apply -f - <<EOF
apiVersion: kubernetes.crossplane.io/v1alpha1
kind: ProviderConfig
metadata:
  name: kubernetes-provider
spec:
  credentials:
    source: InjectedIdentity
EOF
```

### AWS ProviderConfig Setup

Create AWS credentials and configure the provider:

#### Option 1: Using AWS Access Keys

1. Create an AWS IAM user with the following managed policies:
   - `AWSElementalMediaConvertFullAccess`
   - `AWSElementalMediaLiveFullAccess`
   - `AWSElementalMediaPackageFullAccess`
   - `AWSElementalMediaStoreFullAccess`
   - Custom policy for MediaConnect (see below)

2. Create a Kubernetes Secret with AWS credentials:

```bash
kubectl create secret generic provider-aws -n crossplane-system \
  --from-literal=credentials='[default]
aws_access_key_id = YOUR_ACCESS_KEY_ID
aws_secret_access_key = YOUR_SECRET_ACCESS_KEY'
```

3. Create the ProviderConfig:

```bash
kubectl apply -f - <<EOF
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: aws
spec:
  credentials:
    source: Secret
    secretRef:
      namespace: crossplane-system
      name: provider-aws
      key: credentials
EOF
```

#### Option 2: Using IAM Roles for Service Accounts (IRSA) - Recommended for EKS

1. Create an IAM role with the required policies
2. Associate the role with a Kubernetes service account:

```bash
kubectl apply -f - <<EOF
apiVersion: aws.upbound.io/v1beta1
kind: ProviderConfig
metadata:
  name: aws
spec:
  credentials:
    source: InjectedIdentity
EOF
```

### Required IAM Permissions

Create a custom IAM policy for MediaConnect and other services:

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "mediaconnect:*",
        "medialive:*",
        "mediapackage:*",
        "mediapackagev2:*",
        "mediastore:*",
        "mediaconvert:*",
        "mediatailor:*",
        "cloudformation:*",
        "iam:PassRole",
        "iam:CreateRole",
        "iam:CreatePolicy",
        "iam:AttachRolePolicy",
        "iam:ListRoles",
        "iam:GetRole",
        "iam:GetPolicy",
        "ec2:DescribeVpcs",
        "ec2:DescribeSubnets",
        "ec2:DescribeSecurityGroups",
        "ec2:CreateSecurityGroup",
        "ec2:AuthorizeSecurityGroupIngress",
        "logs:CreateLogGroup",
        "logs:CreateLogStream",
        "logs:PutLogEvents"
      ],
      "Resource": "*"
    }
  ]
}
```

## Usage Examples

### Basic MediaLive Network

```yaml
apiVersion: medialive.aws.livewyer.io/v1alpha1
kind: Network
metadata:
  name: my-media-network
spec:
  providerConfigRef:
    name: aws
  forProvider:
    region: us-east-1
    ipPools:
      - Cidr: 192.168.1.0/24
```

### MediaConnect Flow

```yaml
apiVersion: mediaconnect.aws.livewyer.io/v1alpha1
kind: Flow
metadata:
  name: my-media-flow
spec:
  providerConfigRef:
    name: aws
  forProvider:
    region: us-east-1
    source:
      Name: my-source
      Protocol: zixi-push
      IngestPort: 2088
```

### Event-Driven Workflow

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: Event
metadata:
  name: live-streaming-event
spec:
  providerConfigRef:
    name: aws
  forProvider:
    region: us-east-1
  workflowTemplate:
    id: workflow-usecase-1
    parameters:
      network: 192.168.1.0/24
      iamRole: arn:aws:iam::123456789012:role/MediaLiveAccessRole
```

### Workflow Template

```yaml
apiVersion: elemental.aws.livewyer.io/v1alpha1
kind: WorkflowTemplate
metadata:
  name: standard-live-workflow
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
    - name: create-packaging
      resources:
        - name: package-channel
          spec:
            apiVersion: mediapackagev2.aws.livewyer.io/v1alpha1
            kind: ChannelGroup
            spec:
              forProvider:
                channelGroupName: live-content
                description: "Live streaming content group"
```

## Complex Workflow Examples

See the complete workflow examples in the [`examples/`](examples/) directory:

- [`workflow.yaml`](examples/workflow.yaml) - Multi-step workflow with dependencies
- [`workflow-template.yaml`](examples/workflow-template.yaml) - Reusable workflow template
- [`event.yaml`](examples/event.yaml) - Event-driven workflow execution
- Individual service examples in respective subdirectories

## Configuration Structure

```
├── apis/                    # Composite Resource Definitions
│   ├── event/              # Event-driven workflow XRDs
│   ├── mediaconnect/       # MediaConnect XRDs
│   │   ├── bridge/         # Bridge resource
│   │   ├── flow/           # Flow resource
│   │   └── gateway/        # Gateway resource
│   ├── mediaconvert/       # MediaConvert XRDs
│   │   └── jobtemplate/    # JobTemplate resource
│   ├── medialive/          # MediaLive XRDs
│   │   ├── multiplexprogram/ # MultiplexProgram resource
│   │   └── network/        # Network resource
│   ├── mediapackage/       # MediaPackage XRDs
│   │   └── originendpoint/ # OriginEndpoint resource
│   ├── mediapackagev2/     # MediaPackage V2 XRDs
│   │   ├── channel/        # Channel resource
│   │   └── channelgroup/   # ChannelGroup resource
│   ├── mediastore/         # MediaStore XRDs (planned)
│   ├── mediatailor/        # MediaTailor XRDs (planned)
│   ├── workflow/           # Workflow orchestration XRDs
│   └── workflowtemplate/   # WorkflowTemplate XRDs
├── docs/                   # Documentation
├── examples/               # Usage examples
├── functions/              # Composition functions
└── tests/                  # Test configurations
```

## Dependencies

This configuration automatically installs the following dependencies:

### Providers

- `xpkg.upbound.io/upbound/provider-family-aws` (>=v1)
- `xpkg.upbound.io/upbound/provider-aws-cloudcontrol` (>=v1)
- `xpkg.upbound.io/upbound/provider-aws-medialive` (>=v1)
- `xpkg.upbound.io/upbound/provider-aws-mediapackage` (>=v1)
- `xpkg.upbound.io/upbound/provider-aws-mediastore` (>=v1)
- `xpkg.upbound.io/upbound/provider-aws-cloudformation` (>=v1)
- `xpkg.upbound.io/crossplane-contrib/provider-nop` (>=v0.4.0)
- `xpkg.upbound.io/upbound/provider-kubernetes` (>=v0)

### Functions

- `xpkg.upbound.io/upbound/function-go-templating` (>=v0.10.0)
- `xpkg.upbound.io/upbound/function-auto-ready` (>=v0.5.0)

## Documentation

Comprehensive documentation is available in the [`docs/`](docs/) directory:

- [AWS Elemental Resources Overview](docs/README.md)
- [MediaConnect Resources](docs/)
  - [Bridge](docs/mediaconnect-bridge.md)
  - [Flow](docs/mediaconnect-flow.md)
  - [Gateway](docs/mediaconnect-gateway.md)
- [MediaConvert Resources](docs/)
  - [JobTemplate](docs/mediaconvert-jobtemplate.md)
- [MediaLive Resources](docs/)
  - [MultiplexProgram](docs/medialive-multiplexprogram.md)
  - [Network](docs/medialive-network.md)
- [MediaPackage Resources](docs/)
  - [OriginEndpoint](docs/mediapackage-originendpoint.md)
- [MediaPackage V2 Resources](docs/)
  - [Channel](docs/mediapackagev2-channel.md)
  - [ChannelGroup](docs/mediapackagev2-channelgroup.md)
- [Workflow Orchestration](docs/)
  - [Event](docs/event.md)
  - [WorkflowTemplate](docs/workflowtemplate.md)
  - [Workflow](docs/workflow.md)

## Troubleshooting

### Common Issues

1. **Provider not ready**: Wait for all providers to be installed and healthy

   ```bash
   kubectl get providers
   ```

2. **Missing permissions**: Ensure your AWS credentials have all required permissions

3. **Region availability**: Some AWS Elemental services are not available in all regions

4. **Resource dependencies**: Check that dependent resources are created in the correct order

5. **Template parameters**: Ensure all required parameters are provided when using workflow templates

### Debugging

Enable debug logging for providers:

```bash
kubectl patch deployment crossplane -n crossplane-system -p '{"spec":{"template":{"spec":{"containers":[{"name":"crossplane","args":["--debug"]}]}}}}'
```

Check resource status:

```bash
kubectl describe <resource-type> <resource-name>
```

Check workflow execution:

```bash
kubectl get events --sort-by='.metadata.creationTimestamp'
kubectl logs -n crossplane-system deployment/crossplane
```

## Use Cases

### Live Streaming Workflows

- Event-driven live streaming setup with automatic resource provisioning
- Multi-camera live events with failover capabilities
- 24/7 live channels with monitoring and alerting

### Video Processing Pipelines

- File-based transcoding workflows for VOD content
- Live transcoding and packaging for ABR streaming
- Content preparation for multiple distribution platforms

### Broadcast Infrastructure

- Traditional broadcast-to-IP workflows
- Cloud-based master control and playout
- Disaster recovery and backup streaming

### Media Supply Chain

- Content acquisition and preparation workflows
- Quality control and compliance checking
- Multi-region content distribution

## Contributing

We welcome contributions! Please see our contributing guidelines and:

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add/update tests and documentation
5. Submit a pull request

### Development Guidelines

- Follow Crossplane composition best practices
- Include comprehensive examples for new resources
- Update documentation for any new features
- Test with multiple AWS regions where applicable
- Ensure proper error handling and status reporting

## Support

For support and questions:

- Create an issue in this repository
- Review existing documentation and examples
- Contact: bowen@livewyer.com

## Roadmap

### Planned Features

- **MediaStore Resources**: Complete implementation of MediaStore containers and policies
- **MediaTailor Resources**: Ad insertion and personalization workflows
- **Enhanced Monitoring**: Built-in CloudWatch integration and alerting
- **Cost Optimization**: Automated cost optimization recommendations
- **Multi-Region**: Enhanced multi-region deployment capabilities

### Current Status

- ✅ MediaConnect (Bridge, Flow, Gateway)
- ✅ MediaConvert (JobTemplate)
- ✅ MediaLive (MultiplexProgram, Network)
- ✅ MediaPackage (OriginEndpoint)
- ✅ MediaPackage V2 (Channel, ChannelGroup)
- ✅ Event-driven workflows
- ✅ Workflow templates
- 🚧 MediaStore (in development)
- 🚧 MediaTailor (in development)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Acknowledgments

- [Crossplane](https://crossplane.io/) for the amazing cloud-native control plane
- [Upbound](https://upbound.io/) for the AWS provider ecosystem
- AWS Elemental team for the comprehensive media services
- The open-source community for contributions and feedback

---

**Maintained by**: [Livewyer](https://livewyer.com)
**Source**: [github.com/livewyer-ops/crossplane-configuration-aws-elemental](https://github.com/livewyer-ops/crossplane-configuration-aws-elemental)
**Package**: [xpkg.upbound.io/livewyer-ops/configuration-aws-elemental](https://marketplace.upbound.io/configurations/livewyer-ops/configuration-aws-elemental)
