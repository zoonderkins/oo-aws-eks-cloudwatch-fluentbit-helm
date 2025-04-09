# Fluent Bit Helm Chart for Amazon CloudWatch

This Helm chart deploys Fluent Bit as a DaemonSet on a Kubernetes cluster for shipping logs to Amazon CloudWatch.

## Prerequisites

- Kubernetes 1.31+
- Helm 3.0+
- AWS credentials with permissions to write to CloudWatch logs
- (Optional) EKS cluster if using the CloudWatch Observability add-on feature

## Installation

### Add the Helm repository

```bash
helm repo add my-repo https://your-helm-repo-url
helm repo update
```

### Install the chart

```bash
# With default configuration
helm install fluent-bit my-repo/fluent-bit

# With custom values
helm install fluent-bit my-repo/fluent-bit \
  --set aws.region=ap-northeast-1 \
  --set aws.clusterName=my-cluster
```

### Installing from local directory

```bash
git clone https://github.com/yourusername/k8s-otel.git
cd k8s-otel/helm
helm install fluent-bit ./fluent-bit \
  --set aws.region=ap-northeast-1 \
  --set aws.clusterName=my-cluster
```

### Enable CloudWatch Observability EKS Add-on

This chart includes options to help with setting up the EKS add-on for CloudWatch Observability:

```bash
# Enable the add-on with a pre-existing IAM role
helm install fluent-bit ./fluent-bit \
  --set aws.region=ap-northeast-1 \
  --set aws.clusterName=my-cluster \
  --set cloudwatchObservability.enabled=true \
  --set cloudwatchObservability.serviceAccountRoleArn=arn:aws:iam::123456789012:role/eksCloudWatchRole

# Or with instructions to create a new IAM role
helm install fluent-bit ./fluent-bit \
  --set aws.region=ap-northeast-1 \
  --set aws.clusterName=my-cluster \
  --set cloudwatchObservability.enabled=true \
  --set cloudwatchObservability.createRole=true
```

After installing the chart with CloudWatch Observability enabled, follow the instructions printed in the NOTES to complete the add-on installation using AWS CLI or eksctl.

## Configuration

The following table lists the configurable parameters of the Fluent Bit chart and their default values.

| Parameter | Description | Default |
|-----------|-------------|---------|
| `aws.region` | AWS region for CloudWatch logs | `ap-northeast-1` |
| `aws.clusterName` | EKS cluster name | `my-eks-cluster` |
| `namespace.name` | Namespace to create for Fluent Bit | `amazon-cloudwatch` |
| `namespace.create` | Whether to create the namespace | `true` |
| `fluentBit.image.repository` | Fluent Bit container image repository | `public.ecr.aws/aws-observability/aws-for-fluent-bit` |
| `fluentBit.image.tag` | Fluent Bit container image tag | `2.32.5.20250327` |
| `fluentBit.image.pullPolicy` | Image pull policy | `Always` |
| `fluentBit.env.readFromHead` | Read logs from the head of files | `Off` |
| `fluentBit.env.readFromTail` | Read logs from the tail of files | `On` |
| `fluentBit.resources` | CPU/Memory resource requests/limits | See values.yaml |
| `fluentBit.serviceAccount.create` | Create service account | `false` |
| `fluentBit.serviceAccount.name` | Service account name | `cloudwatch-agent` |
| `fluentBit.nodeSelector` | Node labels for pod assignment | `kubernetes.io/os: linux` |
| `fluentBit.tolerations` | Tolerations for pod assignment | `[]` |
| `paths.*` | Various paths for logs and state | See values.yaml |

### CloudWatch Observability Add-on Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `cloudwatchObservability.enabled` | Enable CloudWatch Observability Add-on setup | `false` |
| `cloudwatchObservability.serviceAccountRoleArn` | IAM role ARN for CloudWatch agent service account | `""` |
| `cloudwatchObservability.createRole` | Instructions for creating an IAM role | `false` |
| `cloudwatchObservability.rolePolicyArns` | Policy ARNs to attach to the IAM role | `["arn:aws:iam::aws:policy/CloudWatchAgentServerPolicy"]` |
| `cloudwatchObservability.addOnConfig.conflictResolution` | How to resolve conflicts during add-on installation | `"OVERWRITE"` |

## Log Types Collected

The Fluent Bit configuration collects the following logs:

1. **Application Logs** - Container logs from all pods
2. **Dataplane Logs** - Logs from Docker, containerd, kubelet services
3. **Host Logs** - System logs from the host nodes

These logs are sent to the following CloudWatch log groups:

- `/aws/containerinsights/{cluster-name}/application`
- `/aws/containerinsights/{cluster-name}/dataplane`
- `/aws/containerinsights/{cluster-name}/host` 