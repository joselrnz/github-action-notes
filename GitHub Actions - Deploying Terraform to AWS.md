# GitHub Actions - Deploying Terraform to AWS

This guide provides a standardized approach to deploying Terraform infrastructure to AWS using GitHub Actions, incorporating best practices, environment-based configurations, and secure state management.

## Standardized Terraform Deployment Workflow

```yaml
name: Terraform AWS Deployment

on:
  push:
    branches:
      - main
    paths:
      - terraform/**
  pull_request:
    branches:
      - main
    paths:
      - terraform/**
  workflow_dispatch:
    inputs:
      environment:
        description: 'Target environment (e.g., dev, staging, prod)'
        required: true
        default: 'dev'

jobs:
  terraform:
    runs-on: ubuntu-latest
    env:
      AWS_REGION: us-east-1
      ENVIRONMENT: ${{ github.event.inputs.environment }}
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4
      
      - name: Configure AWS Credentials
        uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          region: ${{ env.AWS_REGION }}
      
      - name: Setup Terraform
        uses: hashicorp/setup-terraform@v1
      
      - name: Terraform Init
        run: |
          terraform init \
            -backend-config="bucket=my-terraform-state-bucket" \
            -backend-config="key=${{ env.ENVIRONMENT }}/terraform.tfstate" \
            -backend-config="region=${{ env.AWS_REGION }}"
      
      - name: Terraform Format Check
        run: terraform fmt -check
      
      - name: Terraform Validate
        run: terraform validate
      
      - name: Terraform Plan
        run: terraform plan -var-file="environments/${{ env.ENVIRONMENT }}.tfvars" -out=tfplan
      
      - name: Terraform Apply
        if: github.ref == 'refs/heads/main'
        run: terraform apply -auto-approve tfplan
```

## Best Practices and Standards

### 1. **Environment-Specific Configurations**

- Use `workflow_dispatch` to allow manual triggers with an `environment` input.
- Maintain separate `.tfvars` files for each environment (e.g., `dev.tfvars`, `staging.tfvars`, `prod.tfvars`).

### 2. **State Management**

- Store Terraform state in an S3 bucket to enable collaboration and maintain consistency.
- Use environment-specific state files (`dev/terraform.tfstate`, `staging/terraform.tfstate`, etc.).

### 3. **Secrets and Credentials**

- Store AWS credentials in GitHub Secrets (`AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY`).
- Ensure least privilege access by using IAM roles where possible.

### 4. **Validation and Formatting**

- Always run `terraform fmt -check` and `terraform validate` before applying changes.
- Ensure proper linting and formatting for readability and consistency.

### 5. **Rollback Strategy**

- Always review Terraform plan outputs before applying changes.
- Maintain backup state versions in S3 and use versioning to enable rollbacks.
- Consider implementing a manual approval step before applying to production environments.

### 6. **Security and Compliance**

- Use Terraform security scanning tools like `tfsec` to detect misconfigurations.
- Implement AWS IAM policies with the principle of least privilege.
- Regularly audit Terraform state and resources for compliance.

### 7. **Infrastructure as Code (IaC) Best Practices**

- Modularize Terraform code for reusability.
- Define variables, outputs, and modules in a structured format.
- Use `terraform destroy` carefully in CI/CD pipelines to avoid accidental deletions.

## Notes

- Modify the `AWS_REGION` variable based on your deployment region.
- Ensure the Terraform backend configuration matches your organization's AWS setup.
- Use `terraform workspace` if working with multiple environments within the same infrastructure repository.

