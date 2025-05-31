# 🔥 Terraform Exam Questions & Answers - Complete Guide

## Easy Questions

### 1. What is Terraform and what problem does it solve?

**Answer:**
Terraform is an open-source Infrastructure as Code (IaC) tool developed by HashiCorp that allows you to define and provision infrastructure using a declarative configuration language. 

**Problems it solves:**
- **Manual Infrastructure Management**: Eliminates manual provisioning through GUIs or command-line tools
- **Infrastructure Drift**: Ensures infrastructure matches the desired state defined in code
- **Consistency**: Provides consistent infrastructure across different environments
- **Version Control**: Infrastructure can be versioned, reviewed, and collaborated on like code
- **Multi-Cloud Support**: Works across multiple cloud providers with a unified workflow

### 2. What language is used to write Terraform configuration files?

**Answer:**
Terraform uses **HashiCorp Configuration Language (HCL)**, which is a declarative language designed to be human-readable and machine-friendly. Configuration files typically have the `.tf` extension. Terraform also supports JSON format (`.tf.json` files) as an alternative syntax.

### 3. Name three main Terraform commands and their purpose.

**Answer:**
1. **`terraform init`**: Initializes a Terraform working directory by downloading required providers and modules
2. **`terraform plan`**: Creates an execution plan showing what actions Terraform will take to reach the desired state
3. **`terraform apply`**: Executes the actions proposed in the Terraform plan to create, update, or delete infrastructure

### 4. What is a Terraform provider? Give an example.

**Answer:**
A Terraform provider is a plugin that enables Terraform to interact with APIs of cloud platforms, SaaS providers, and other services. Providers are responsible for understanding API interactions and exposing resources.

**Example:**
```hcl
provider "aws" {
  region = "us-west-2"
}

provider "azurerm" {
  features {}
}
```

Popular providers include AWS, Azure, Google Cloud, Kubernetes, GitHub, and hundreds of others.

### 5. What is the purpose of the Terraform state file?

**Answer:**
The Terraform state file (`terraform.tfstate`) serves several critical purposes:

- **Resource Tracking**: Maps real-world resources to your configuration
- **Metadata Storage**: Stores resource metadata and dependencies
- **Performance**: Caches resource attributes to avoid querying providers repeatedly
- **Collaboration**: Enables team collaboration by providing a single source of truth
- **Change Detection**: Determines what changes need to be made during planning

### 6. How do you define a resource in Terraform? Give a simple example.

**Answer:**
Resources are defined using the `resource` block with the syntax: `resource "PROVIDER_RESOURCE_TYPE" "LOCAL_NAME"`

**Example:**
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0c02fb55956c7d316"
  instance_type = "t2.micro"
  
  tags = {
    Name = "WebServer"
    Environment = "Production"
  }
}
```

### 7. What is the difference between `terraform plan` and `terraform apply`?

**Answer:**
- **`terraform plan`**: 
  - Shows a preview of changes without making them
  - Creates an execution plan
  - Safe to run multiple times
  - No infrastructure changes occur
  - Helps review changes before applying

- **`terraform apply`**: 
  - Actually makes the changes to infrastructure
  - Executes the plan to reach the desired state
  - Prompts for confirmation (unless `-auto-approve` is used)
  - Updates the state file after successful execution
  - Can be destructive if not used carefully

### 8. What is a variable in Terraform and why is it useful?

**Answer:**
Variables in Terraform are parameters that allow you to customize configurations without altering the main code.

**Types of variables:**
- Input variables (`variable` blocks)
- Local values (`locals` blocks)
- Output values (`output` blocks)

**Example:**
```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

resource "aws_instance" "example" {
  instance_type = var.instance_type
}
```

**Benefits:**
- Code reusability across environments
- Parameterization without code duplication
- Separation of configuration from implementation
- Enhanced security (sensitive variables)

## Medium Questions

### 1. Explain the Terraform workflow from writing code to deployment.

**Answer:**
The Terraform workflow follows these stages:

1. **Write Configuration**: Create `.tf` files defining desired infrastructure
2. **Initialize**: Run `terraform init` to download providers and modules
3. **Plan**: Execute `terraform plan` to preview changes
4. **Review**: Examine the execution plan for accuracy
5. **Apply**: Run `terraform apply` to provision infrastructure
6. **Manage**: Use `terraform show`, `terraform state list` to inspect resources
7. **Update**: Modify configuration and repeat the cycle
8. **Destroy**: Use `terraform destroy` when infrastructure is no longer needed

**Detailed workflow:**
```bash
# 1. Initialize
terraform init

# 2. Validate syntax
terraform validate

# 3. Format code
terraform fmt

# 4. Plan changes
terraform plan -out=tfplan

# 5. Apply changes
terraform apply tfplan

# 6. Verify deployment
terraform show
```

### 2. How does Terraform handle changes to infrastructure when you update your `.tf` files?

**Answer:**
Terraform uses a **declarative approach** with **state comparison**:

1. **State Comparison**: Compares current state file with desired configuration
2. **Dependency Graph**: Creates a dependency graph to determine order of operations
3. **Change Detection**: Identifies resources to create, update, or delete
4. **Execution Plan**: Shows exactly what will change during `terraform plan`
5. **Resource Lifecycle**: Handles different types of changes:
   - **Create**: New resources not in state
   - **Update**: In-place updates when possible
   - **Replace**: Destroy and recreate when in-place update isn't possible
   - **Delete**: Remove resources not in configuration

**Example of change types:**
```hcl
# Original
resource "aws_instance" "example" {
  instance_type = "t2.micro"  # This can be updated in-place
  ami          = "ami-12345"  # This requires replacement
}

# Updated - will show update and replace operations
resource "aws_instance" "example" {
  instance_type = "t3.micro"  # Update in-place
  ami          = "ami-67890"  # Force replacement
}
```

### 3. What are Terraform modules and why should you use them?

**Answer:**
Modules are containers for multiple resources that are used together. They're the main way to package and reuse resource configurations.

**Types of modules:**
- **Root Module**: The working directory where you run Terraform commands
- **Child Module**: Modules called by other modules
- **Published Modules**: Modules shared via Terraform Registry

**Module structure:**
```
module/
├── main.tf
├── variables.tf
├── outputs.tf
└── README.md
```

**Benefits:**
- **Reusability**: Write once, use multiple times
- **Organization**: Logical grouping of resources
- **Abstraction**: Hide complexity behind simple interfaces
- **Consistency**: Standardized infrastructure patterns
- **Collaboration**: Share modules across teams

**Example usage:**
```hcl
module "vpc" {
  source = "./modules/vpc"
  
  cidr_block = "10.0.0.0/16"
  environment = "production"
}
```

### 4. What is the difference between a resource and a data source in Terraform?

**Answer:**

| Aspect | Resource | Data Source |
|--------|----------|-------------|
| **Purpose** | Creates/manages infrastructure | Fetches existing infrastructure info |
| **Keyword** | `resource` | `data` |
| **State Management** | Tracked in state file | Not tracked in state |
| **Lifecycle** | Can be created, updated, destroyed | Read-only |
| **Usage** | Define new infrastructure | Query existing infrastructure |

**Examples:**

```hcl
# RESOURCE - Creates new infrastructure
resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
}

# DATA SOURCE - Queries existing infrastructure
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-focal-20.04-amd64-server-*"]
  }
}

# DATA SOURCE - Get existing VPC
data "aws_vpc" "existing" {
  filter {
    name   = "tag:Name"
    values = ["existing-vpc"]
  }
}
```

### 5. Describe remote state in Terraform. Why is it important?

**Answer:**
Remote state stores the Terraform state file in a remote location instead of locally.

**Remote State Backends:**
- **S3**: AWS S3 bucket with DynamoDB for locking
- **Azure Storage**: Azure Storage Account
- **Google Cloud Storage**: GCS bucket
- **Terraform Cloud**: HashiCorp's managed service
- **Consul**: HashiCorp Consul
- **HTTP**: Custom HTTP endpoint

**Configuration example:**
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-locks"
    encrypt        = true
  }
}
```

**Why it's important:**
- **Team Collaboration**: Multiple team members can access the same state
- **State Locking**: Prevents concurrent modifications
- **Security**: State files often contain sensitive information
- **Reliability**: Backup and versioning capabilities
- **Consistency**: Single source of truth for infrastructure state

### 6. How do you manage multiple environments (like dev, staging, prod) in Terraform?

**Answer:**
There are several approaches to manage multiple environments:

#### Approach 1: Workspaces
```bash
# Create workspaces
terraform workspace new dev
terraform workspace new staging
terraform workspace new prod

# Switch between workspaces
terraform workspace select prod
terraform apply
```

#### Approach 2: Separate Directories
```
environments/
├── dev/
│   ├── main.tf
│   └── terraform.tfvars
├── staging/
│   ├── main.tf
│   └── terraform.tfvars
└── prod/
    ├── main.tf
    └── terraform.tfvars
```

#### Approach 3: Variable Files
```bash
# Different variable files per environment
terraform apply -var-file="dev.tfvars"
terraform apply -var-file="prod.tfvars"
```

**Best Practices:**
- Use modules for shared infrastructure patterns
- Implement different variable values per environment
- Use separate state files/backends per environment
- Apply environment-specific resource sizing and configurations

### 7. Explain how you can protect a resource from accidental deletion in Terraform.

**Answer:**
Terraform provides several mechanisms to protect resources:

#### 1. Lifecycle Rules
```hcl
resource "aws_instance" "critical" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  
  lifecycle {
    prevent_destroy = true
  }
}
```

#### 2. Termination Protection (AWS-specific)
```hcl
resource "aws_instance" "protected" {
  ami                     = "ami-12345"
  instance_type          = "t2.micro"
  disable_api_termination = true
}
```

#### 3. Resource Dependencies
```hcl
resource "aws_eip" "example" {
  instance = aws_instance.web.id
  
  lifecycle {
    create_before_destroy = true
  }
}
```

#### 4. State File Backup
```bash
# Backup state before destructive operations
cp terraform.tfstate terraform.tfstate.backup
```

#### 5. Import Protection
```hcl
# Use data sources instead of managing existing critical resources
data "aws_instance" "existing_critical" {
  instance_id = "i-1234567890abcdef0"
}
```

### 8. How can you use provisioners in Terraform? Give a use case.

**Answer:**
Provisioners are used to execute scripts or commands on local or remote machines as part of resource creation or destruction.

**Types of provisioners:**
- **local-exec**: Executes commands locally
- **remote-exec**: Executes commands remotely
- **file**: Copies files/directories

**Use case example - Configure web server:**
```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345"
  instance_type = "t2.micro"
  key_name      = "my-key"
  
  # Copy configuration file
  provisioner "file" {
    source      = "web-config.conf"
    destination = "/tmp/web-config.conf"
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
  
  # Install and configure web server
  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo cp /tmp/web-config.conf /etc/nginx/nginx.conf",
      "sudo systemctl start nginx",
      "sudo systemctl enable nginx"
    ]
    
    connection {
      type        = "ssh"
      user        = "ubuntu"
      private_key = file("~/.ssh/id_rsa")
      host        = self.public_ip
    }
  }
  
  # Local notification
  provisioner "local-exec" {
    command = "echo 'Web server ${self.public_ip} is ready!'"
  }
}
```

**Best Practices:**
- Use provisioners as a last resort
- Prefer cloud-init, user data, or configuration management tools
- Always include proper error handling
- Use null_resource for provisioner-only resources

## Hard Questions

### 1. Describe the process of creating a reusable Terraform module. What should be considered while designing one?

**Answer:**
Creating a reusable Terraform module involves careful planning and design considerations.

#### Module Structure:
```
modules/vpc/
├── main.tf          # Main resource definitions
├── variables.tf     # Input variables
├── outputs.tf       # Output values
├── versions.tf      # Provider version constraints
├── README.md        # Documentation
└── examples/        # Usage examples
    └── basic/
        ├── main.tf
        └── variables.tf
```

#### Design Considerations:

**1. Interface Design:**
```hcl
# variables.tf - Well-defined inputs
variable "vpc_cidr" {
  description = "CIDR block for VPC"
  type        = string
  validation {
    condition     = can(cidrhost(var.vpc_cidr, 0))
    error_message = "VPC CIDR must be a valid IPv4 CIDR block."
  }
}

variable "availability_zones" {
  description = "List of availability zones"
  type        = list(string)
  default     = []
}

variable "tags" {
  description = "Tags to apply to resources"
  type        = map(string)
  default     = {}
}
```

**2. Flexible Configuration:**
```hcl
# main.tf - Flexible resource creation
resource "aws_vpc" "main" {
  cidr_block           = var.vpc_cidr
  enable_dns_hostnames = var.enable_dns_hostnames
  enable_dns_support   = var.enable_dns_support
  
  tags = merge(
    var.tags,
    {
      Name = var.vpc_name
    }
  )
}

# Conditional resource creation
resource "aws_internet_gateway" "main" {
  count  = var.create_igw ? 1 : 0
  vpc_id = aws_vpc.main.id
  
  tags = merge(var.tags, { Name = "${var.vpc_name}-igw" })
}
```

**3. Clear Outputs:**
```hcl
# outputs.tf - Useful outputs for consumers
output "vpc_id" {
  description = "ID of the VPC"
  value       = aws_vpc.main.id
}

output "private_subnet_ids" {
  description = "List of private subnet IDs"
  value       = aws_subnet.private[*].id
}

output "public_subnet_ids" {
  description = "List of public subnet IDs"
  value       = aws_subnet.public[*].id
}
```

**4. Version Constraints:**
```hcl
# versions.tf
terraform {
  required_version = ">= 0.15"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = ">= 4.0"
    }
  }
}
```

**Key Design Principles:**
- **Single Responsibility**: Each module should have a clear, focused purpose
- **Composability**: Modules should work well together
- **Configurability**: Provide sensible defaults but allow customization
- **Documentation**: Clear README with examples and parameter descriptions
- **Validation**: Input validation to catch errors early
- **Backward Compatibility**: Semantic versioning for module releases

### 2. Explain the importance of state locking and how Terraform achieves it.

**Answer:**
State locking prevents multiple Terraform processes from running simultaneously against the same state, which could corrupt the state file or cause race conditions.

#### Why State Locking is Critical:
- **Data Integrity**: Prevents state file corruption
- **Consistency**: Ensures only one operation modifies infrastructure at a time
- **Team Safety**: Prevents conflicting changes in team environments
- **Operation Safety**: Avoids partial updates and inconsistent states

#### How Terraform Achieves State Locking:

**1. Backend-Specific Locking:**
```hcl
# S3 backend with DynamoDB locking
terraform {
  backend "s3" {
    bucket         = "terraform-state-bucket"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    
    # DynamoDB table for state locking
    dynamodb_table = "terraform-state-locks"
  }
}
```

**2. DynamoDB Table Structure:**
```hcl
resource "aws_dynamodb_table" "terraform_locks" {
  name           = "terraform-state-locks"
  billing_mode   = "PAY_PER_REQUEST"
  hash_key       = "LockID"

  attribute {
    name = "LockID"
    type = "S"
  }

  tags = {
    Name = "Terraform State Lock Table"
  }
}
```

**3. Lock Information:**
When locked, DynamoDB stores:
- **LockID**: Unique identifier for the lock
- **Info**: Information about who acquired the lock
- **Created**: When the lock was acquired
- **Operation**: What operation is being performed

**4. Lock Behavior:**
```bash
# Terraform automatically acquires lock during:
terraform plan    # Acquires read lock
terraform apply   # Acquires write lock
terraform destroy # Acquires write lock

# Manual lock operations:
terraform force-unlock LOCK_ID  # Emergency unlock (use carefully)
```

**5. Different Backend Locking Support:**
- **S3 + DynamoDB**: Full locking support
- **Azure Storage**: Native locking support
- **GCS**: Native locking support
- **Terraform Cloud**: Built-in locking
- **Local**: File-based locking
- **Consul**: Distributed locking

**Best Practices:**
- Always use backends that support locking in team environments
- Monitor lock timeouts and failed operations
- Have a process for handling stuck locks
- Use CI/CD pipelines to serialize Terraform operations

### 3. How can you integrate Terraform with CI/CD pipelines?

**Answer:**
Integrating Terraform with CI/CD pipelines enables automated infrastructure deployment and management.

#### CI/CD Pipeline Stages:

**1. Pipeline Structure:**
```yaml
# .github/workflows/terraform.yml (GitHub Actions example)
name: Terraform CI/CD

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

env:
  TF_VERSION: "1.5.0"
  AWS_REGION: "us-west-2"

jobs:
  plan:
    runs-on: ubuntu-latest
    if: github.event_name == 'pull_request'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: ${{ env.TF_VERSION }}
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Validate
      run: terraform validate
    
    - name: Terraform Plan
      run: terraform plan -no-color
      continue-on-error: true
    
    - name: Comment PR
      uses: actions/github-script@v6
      with:
        script: |
          const output = `#### Terraform Plan 📖
          \`\`\`
          ${{ steps.plan.outputs.stdout }}
          \`\`\``;
          github.rest.issues.createComment({
            issue_number: context.issue.number,
            owner: context.repo.owner,
            repo: context.repo.repo,
            body: output
          })

  apply:
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main' && github.event_name == 'push'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
      with:
        terraform_version: ${{ env.TF_VERSION }}
    
    - name: Configure AWS credentials
      uses: aws-actions/configure-aws-credentials@v2
      with:
        aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
        aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
        aws-region: ${{ env.AWS_REGION }}
    
    - name: Terraform Init
      run: terraform init
    
    - name: Terraform Plan
      run: terraform plan -out=tfplan
    
    - name: Terraform Apply
      run: terraform apply -auto-approve tfplan
```

**2. Jenkins Pipeline Example:**
```groovy
pipeline {
    agent any
    
    environment {
        AWS_DEFAULT_REGION = 'us-west-2'
        TF_VAR_environment = "${env.BRANCH_NAME}"
    }
    
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        
        stage('Terraform Init') {
            steps {
                sh 'terraform init -backend-config="key=env/${BRANCH_NAME}/terraform.tfstate"'
            }
        }
        
        stage('Terraform Plan') {
            steps {
                script {
                    def planOutput = sh(
                        script: 'terraform plan -out=tfplan -no-color',
                        returnStdout: true
                    ).trim()
                    
                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: '.',
                        reportFiles: 'plan.html',
                        reportName: 'Terraform Plan'
                    ])
                }
            }
        }
        
        stage('Approval') {
            when {
                branch 'main'
            }
            steps {
                input message: 'Apply Terraform changes?', ok: 'Apply'
            }
        }
        
        stage('Terraform Apply') {
            when {
                branch 'main'
            }
            steps {
                sh 'terraform apply -auto-approve tfplan'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
        failure {
            emailext (
                subject: "Terraform Pipeline Failed: ${env.JOB_NAME} - ${env.BUILD_NUMBER}",
                body: "Pipeline failed. Check logs: ${env.BUILD_URL}",
                to: "${env.CHANGE_AUTHOR_EMAIL}"
            )
        }
    }
}
```

**3. Best Practices for CI/CD Integration:**

**Security:**
```yaml
# Use secrets management
- name: Configure credentials
  env:
    AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
    AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
    TF_VAR_db_password: ${{ secrets.DB_PASSWORD }}
```

**Environment Management:**
```bash
# Use workspaces or separate backends
terraform workspace select ${ENVIRONMENT}
terraform init -backend-config="key=${ENVIRONMENT}/terraform.tfstate"
```

**Quality Gates:**
```yaml
- name: Security Scan
  run: |
    # Use tools like tfsec, checkov, or terrascan
    tfsec .
    checkov -d . --framework terraform

- name: Cost Estimation
  run: |
    # Use infracost for cost estimation
    infracost breakdown --path .
```

**Rollback Strategy:**
```bash
# Plan-based rollback
terraform plan -destroy -out=destroy.tfplan
terraform apply destroy.tfplan

# State-based rollback
terraform state pull > backup.tfstate
# Restore previous state if needed
```

### 4. What challenges can arise from Terraform state file conflicts and how do you mitigate them?

**Answer:**
State file conflicts are one of the most critical challenges in Terraform, especially in team environments.

#### Common State Conflict Scenarios:

**1. Concurrent Modifications:**
```bash
# Team member A runs:
terraform apply  # Acquires lock

# Team member B tries to run simultaneously:
terraform apply  # Gets lock error
```

**2. Manual Infrastructure Changes:**
```bash
# Someone manually deletes a resource in AWS console
terraform plan   # Shows resource needs to be created
terraform apply  # May fail or create duplicate resources
```

**3. State Drift:**
```bash
# Infrastructure changed outside Terraform
terraform plan   # Shows unexpected changes
```

**4. Import Conflicts:**
```bash
# Trying to import already managed resource
terraform import aws_instance.web i-1234567890abcdef0
# Error: resource already exists in state
```

#### Mitigation Strategies:

**1. Remote State with Locking:**
```hcl
terraform {
  backend "s3" {
    bucket         = "terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true
    dynamodb_table = "terraform-locks"  # Critical for locking
  }
}
```

**2. State Management Commands:**
```bash
# Refresh state to detect drift
terraform refresh

# Show current state
terraform show

# List resources in state
terraform state list

# Remove resource from state without destroying
terraform state rm aws_instance.example

# Move resource in state
terraform state mv aws_instance.old aws_instance.new

# Import existing resource
terraform import aws_instance.web i-1234567890abcdef0
```

**3. Conflict Resolution Process:**
```bash
# 1. Backup current state
terraform state pull > backup-$(date +%Y%m%d_%H%M%S).tfstate

# 2. Identify conflicts
terraform plan -detailed-exitcode

# 3. Resolve conflicts based on type:

# For drift detection:
terraform refresh
terraform plan

# For import conflicts:
terraform state rm resource.name
terraform import resource.name existing-id

# For duplicate resources:
terraform state show resource.name
# Manually resolve in AWS console or state file
```

**4. Prevention Best Practices:**

**Team Workflows:**
```yaml
# .github/workflows/terraform-pr.yml
name: Terraform PR Check
on:
  pull_request:
    paths: ['terraform/**']

jobs:
  plan:
    runs-on: ubuntu-latest
    steps:
    - name: Checkout
      uses: actions/checkout@v3
    
    - name: Setup Terraform
      uses: hashicorp/setup-terraform@v2
    
    - name: Terraform Plan
      run: |
        cd terraform
        terraform init
        terraform plan -lock-timeout=10m
```

**Monitoring and Alerting:**
```bash
# Monitor state file changes
aws s3api head-object \
  --bucket terraform-state \
  --key prod/terraform.tfstate \
  --query LastModified

# Alert on unexpected changes
terraform plan -detailed-exitcode
if [ $? -eq 2 ]; then
  echo "Unexpected infrastructure drift detected!"
  # Send alert
fi
```

**5. Advanced State Management:**

**State Splitting:**
```bash
# Split large state files
terraform state mv aws_instance.web module.ec2.aws_instance.web
```

**State Backup Strategy:**
```bash
# Automated backups before operations
#!/bin/bash
BACKUP_DIR="state-backups/$(date +%Y%m%d)"
mkdir -p $BACKUP_DIR
terraform state pull > $BACKUP_DIR/terraform.tfstate.backup
```

**Multi-Environment State Isolation:**
```hcl
# Separate backends per environment
terraform {
  backend "s3" {
    bucket = "terraform-state-${var.environment}"
    key    = "terraform.tfstate"
    region = "us-west-2"
  }
}
```

### 5. Explain lifecycle rules in Terraform and provide scenarios where `create_before_destroy` is necessary.

**Answer:**
Lifecycle rules in Terraform control how resources are created, updated, and destroyed. They provide fine-grained control over resource management behavior.

#### Available Lifecycle Rules:

**1. `create_before_destroy`:**
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = "t3.micro"
  
  lifecycle {
    create_before_destroy = true
  }
}
```

**2. `prevent_destroy`:**
```hcl
resource "aws_s3_bucket" "important_data" {
  bucket = "critical-business-data"
  
  lifecycle {
    prevent_destroy = true
  }
}
```

**3. `ignore_changes`:**
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  lifecycle {
    ignore_changes = [
      ami,           # Ignore AMI changes
      user_data,     # Ignore user data changes
      tags["LastModified"]  # Ignore specific tag changes
    ]
  }
}
```

**4. `replace_triggered_by`:**
```hcl
resource "aws_instance" "web" {
  ami           = var.ami_id
  instance_type = var.instance_type
  
  lifecycle {
    replace_triggered_by = [
      null_resource.cluster_config
    ]
  }
}
```

#### Scenarios Where `create_before_destroy` is Necessary:

**1. Load Balancer Target Groups:**
```hcl
resource "aws_lb_target_group" "web" {
  name     = "web-tg"
  port     = 80
  protocol =