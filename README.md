# GitHub Actions Platform

Shared GitHub Actions for ECS GitOps deployments. This repository provides reusable composite actions for building Docker images, pushing to ECR, and deploying to ECS.

## Actions Available

### build-push-ecr

Build a Docker image and push it to Amazon ECR.

### deploy-ecs

Update an ECS service with a new container image.

## Usage

### Build and Push to ECR

```yaml
- name: Build and push
  uses: akorshak-afg/github-actions-platform/actions/build-push-ecr@v1
  with:
    aws-account-id: ${{ secrets.OPERATIONS_ACCOUNT_ID }}
    ecr-repo-name: my-app
    image-tag: ${{ github.sha }}
    role-to-assume: arn:aws:iam::${{ secrets.OPERATIONS_ACCOUNT_ID }}:role/github-actions-ecr-push
```

### Deploy to ECS

```yaml
- name: Deploy to ECS
  uses: akorshak-afg/github-actions-platform/actions/deploy-ecs@v1
  with:
    aws-account-id: ${{ secrets.DEV_ACCOUNT_ID }}
    cluster: my-cluster
    service: my-service
    container-name: app
    image-uri: ${{ steps.build.outputs.image-uri }}
    role-to-assume: arn:aws:iam::${{ secrets.DEV_ACCOUNT_ID }}:role/github-actions-deploy-dev
```

## Inputs/Outputs Reference

### build-push-ecr

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `aws-account-id` | AWS account ID for ECR | Yes | - |
| `aws-region` | AWS region | No | `us-east-1` |
| `ecr-repo-name` | ECR repository name | Yes | - |
| `image-tag` | Image tag to use | Yes | - |
| `role-to-assume` | IAM role ARN to assume | Yes | - |
| `dockerfile` | Path to Dockerfile | No | `Dockerfile` |
| `context` | Build context path | No | `.` |

#### Outputs

| Name | Description |
|------|-------------|
| `image-uri` | Full ECR image URI |

### deploy-ecs

#### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `aws-account-id` | AWS account ID | Yes | - |
| `aws-region` | AWS region | No | `us-east-1` |
| `cluster` | ECS cluster name | Yes | - |
| `service` | ECS service name | Yes | - |
| `container-name` | Container name to update | Yes | - |
| `image-uri` | New image URI | Yes | - |
| `role-to-assume` | IAM role ARN to assume | Yes | - |
| `wait-for-stable` | Wait for service to stabilize | No | `true` |
| `wait-timeout-seconds` | Timeout for stability wait | No | `300` |

#### Outputs

| Name | Description |
|------|-------------|
| `task-definition-arn` | New task definition ARN |

## Versioning Strategy

This repository uses semantic versioning. When referencing actions in your workflows:

- **Recommended**: Use major version tags (e.g., `@v1`) for stability
- **Specific version**: Use full version tags (e.g., `@v1.0.0`) for reproducibility
- **Latest**: Use `@main` for the latest changes (not recommended for production)

## Development

### Testing Changes

1. Make your changes to the action files
2. Push to a branch
3. The self-test workflow will run automatically on push to main
4. For branch testing, trigger the workflow manually

### Releasing New Versions

```bash
# Create a new version tag
git tag v1.0.1
git push origin v1.0.1

# Update the major version tag (for @v1 users)
git tag -f v1
git push origin v1 --force
```

## Requirements

- GitHub Actions runner with Docker support
- AWS OIDC provider configured for GitHub Actions
- IAM roles with appropriate permissions for ECR and ECS operations

## License

MIT
