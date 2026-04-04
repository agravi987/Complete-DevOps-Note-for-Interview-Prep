# Terraform HCL Syntax and Variables

## 1. Overview
Terraform uses **HCL (HashiCorp Configuration Language)** — a declarative, human-readable language designed specifically for infrastructure configuration. Understanding HCL syntax and Terraform's variable system is essential for writing reusable, flexible, and environment-agnostic infrastructure code.

## 2. Why This Matters
- **Reusability:** Variables allow the same Terraform module to deploy to `dev`, `staging`, and `production` with different configurations — no copy-pasting code.
- **Security:** Sensitive values (passwords, API keys) are passed in via variables at runtime instead of being hardcoded in `.tf` files checked into Git.
- **Readability:** Outputs expose resource attributes (like an EC2 public IP or S3 bucket name) so other modules or users can easily consume them.

## 3. Core Concepts

### HCL Building Blocks
- **Block:** The structural unit in HCL. Contains a *type*, optional *labels*, and a *body*.
  ```hcl
  resource "aws_instance" "web" {    # block_type "label1" "label2"
    ami = "ami-12345"                 # argument = value
  }
  ```
- **Arguments:** Key-value pairs inside a block body (`ami = "ami-12345"`).
- **Expressions:** Dynamic values — can be literals, references, or function calls.

### Variable Types
- **Input Variables (`variable`):** Parameters passed INTO a Terraform configuration. Like function arguments.
- **Local Values (`locals`):** Computed, reusable constants within a module. Like local variables in a function.
- **Output Values (`output`):** Values exposed FROM a Terraform configuration. Like return values.
- **Data Sources (`data`):** Read-only queries of existing infrastructure or external data (e.g., look up the latest Ubuntu AMI ID dynamically).

### Variable Types (HCL Types)
| Type | Example |
|---|---|
| `string` | `"ap-south-1"` |
| `number` | `3` |
| `bool` | `true` |
| `list(string)` | `["us-east-1a", "us-east-1b"]` |
| `map(string)` | `{ env = "prod", team = "devops" }` |
| `object({})` | Complex structured type |

## 4. Architecture / Workflow

```
variables.tf       →  Declares what parameters the module accepts
terraform.tfvars   →  Provides actual values for those parameters (per environment)
locals.tf          →  Computes derived values from variables
main.tf            →  Uses variables/locals to build resources
outputs.tf         →  Exposes important values for other modules to consume
```

## 5. Commands & Practical Usage

Pass a variable value directly on the CLI:
```bash
terraform apply -var="instance_type=t3.large"
```

Use a dedicated variable file for an environment:
```bash
terraform apply -var-file="production.tfvars"
```

List all outputs after an apply:
```bash
terraform output
# Get a specific output value for scripting
terraform output -raw ec2_public_ip
```

## 6. Configuration / Code Examples

**`variables.tf`** — Declare all input variables with types and defaults:
```hcl
variable "aws_region" {
  description = "AWS region to deploy resources in"
  type        = string
  default     = "ap-south-1"
}

variable "environment" {
  description = "Deployment environment (dev, staging, prod)"
  type        = string

  validation {
    condition     = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be one of: dev, staging, prod."
  }
}

variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}

variable "db_password" {
  description = "Database master password"
  type        = string
  sensitive   = true  # Terraform will redact this from logs and plan output
}
```

**`locals.tf`** — Compute reusable derived values:
```hcl
locals {
  # Construct a standard name prefix used across all resources
  name_prefix = "${var.environment}-myapp"

  # Common tags applied to every resource automatically
  common_tags = {
    Environment = var.environment
    ManagedBy   = "terraform"
    Project     = "my-devops-app"
  }
}
```

**`main.tf`** — Reference variables and locals:
```hcl
resource "aws_instance" "app_server" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = var.instance_type

  tags = merge(local.common_tags, {
    Name = "${local.name_prefix}-server"
  })
}

# Data source: Dynamically look up the latest Ubuntu 22.04 AMI
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical's AWS Account ID

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
}
```

**`outputs.tf`** — Expose values:
```hcl
output "server_public_ip" {
  description = "The public IP address of the application server"
  value       = aws_instance.app_server.public_ip
}

output "server_id" {
  description = "The EC2 instance ID"
  value       = aws_instance.app_server.id
}
```

**`production.tfvars`** — Environment-specific values:
```hcl
aws_region    = "us-east-1"
environment   = "prod"
instance_type = "m5.large"
# db_password is passed via environment variable: TF_VAR_db_password
```

## 7. Hands-on Step-by-Step (VERY IMPORTANT)

**Step 1: Create a variables-driven project structure**
```bash
mkdir tf-variables-lab && cd tf-variables-lab
touch main.tf variables.tf outputs.tf dev.tfvars prod.tfvars
```

**Step 2: Populate `variables.tf`**
```hcl
variable "bucket_name" {
  type        = string
  description = "Name of the S3 bucket"
}

variable "enable_versioning" {
  type    = bool
  default = false
}
```

**Step 3: Populate `main.tf`**
```hcl
terraform {
  required_providers {
    aws = { source = "hashicorp/aws", version = "~> 5.0" }
  }
}

provider "aws" { region = "ap-south-1" }

resource "aws_s3_bucket" "this" {
  bucket = var.bucket_name
}

resource "aws_s3_bucket_versioning" "this" {
  bucket = aws_s3_bucket.this.id
  versioning_configuration {
    status = var.enable_versioning ? "Enabled" : "Suspended"
  }
}
```

**Step 4: Populate `outputs.tf`**
```hcl
output "bucket_arn" {
  value = aws_s3_bucket.this.arn
}
```

**Step 5: Create `dev.tfvars`**
```hcl
bucket_name       = "my-app-dev-bucket-2026"
enable_versioning = false
```

**Step 6: Deploy to dev environment**
```bash
terraform init
terraform apply -var-file="dev.tfvars"
terraform output bucket_arn
```

## 8. Common Errors & Troubleshooting

- **Error: `Variable not set` / `No value for required variable`**
  - **Issue:** A variable with no `default` was not provided a value via `-var`, `-var-file`, env variable, or interactive prompt.
  - **Fix:** Provide it via `terraform apply -var="name=value"`, add it to a `.tfvars` file, or set `TF_VAR_name=value` shell environment variable.

- **Error: `Invalid value for variable` (validation block failure)**
  - **Issue:** The value passed for a variable fails its `validation` condition (e.g., passing `"staging2"` when only `dev/staging/prod` are allowed).
  - **Fix:** Pass one of the accepted values as defined in the validation block's `condition`.

- **Error: Output value is `(sensitive value)` and you can't see it**
  - **Issue:** The output references a resource attribute that Terraform marks as sensitive.
  - **Fix:** Read it via `terraform output -raw <name>` for direct use in scripts, or mark it explicitly in the output block: `sensitive = true`.

## 9. Best Practices

1. **Never hardcode environment-specific values in `main.tf`.** Always use variables so the same code targets different environments.
2. **Mark sensitive variables with `sensitive = true`.** This redacts the values from `terraform plan` and `terraform apply` terminal output, preventing accidental credential exposure in CI/CD logs.
3. **Use `locals` to avoid repetition.** If you find yourself writing the same string expression in 5 places, compute it once in a `locals` block and reference it everywhere.
4. **Use `validation` blocks for input variables.** Catch invalid input early with a clear error message rather than waiting for an obscure AWS API error.

## 10. Interview Questions & Answers

**Q1: What is the difference between `variable`, `local`, and `output` in Terraform?**
**A1:** `variable` declares an **input parameter** for a module (passed in from outside). `local` declares a **computed constant** used internally within the module. `output` declares a **return value** exposed from the module for other modules or users to consume.

**Q2: How do you pass a secret (like a database password) to Terraform without hardcoding it?**
**A2:** Declare the variable as `sensitive = true` in `variables.tf`. Then provide the value at runtime via: environment variable (`TF_VAR_db_password=secret`), the interactive prompt (if no value is provided, Terraform will prompt), or a secrets manager integration like AWS Secrets Manager via a `data` source.

**Q3: What is a `data` source in Terraform?**
**A3:** A `data` source lets Terraform **query existing infrastructure or external APIs** read-only — without managing or modifying those resources. A classic use case is dynamically looking up the latest Ubuntu AMI ID so your code never has a hardcoded, stale AMI ID.

**Q4: What is the `merge()` function used for in HCL?**
**A4:** `merge()` combines two or more maps into one. It's commonly used to merge `common_tags` (containing `Environment`, `ManagedBy`) with resource-specific tags (containing `Name`), producing a complete tag set for a resource cleanly.

**Q5: If you have a `production.tfvars` file, how does Terraform know to use it?**
**A5:** Terraform does NOT automatically load `.tfvars` files (except `terraform.tfvars` and `*.auto.tfvars`). You must explicitly reference it: `terraform apply -var-file="production.tfvars"`.

## 11. Quick Revision Summary
- **HCL:** Human-readable blocks, arguments, and expressions.
- **`variable`:** Inputs. Define type, default, validation. Inject at runtime.
- **`local`:** Reusable computed values. DRY principle for HCL.
- **`output`:** Return values exposed to users and other modules.
- **`data`:** Read-only queries of existing resources.

## 12. Official Documentation Links
- [Terraform Input Variables](https://developer.hashicorp.com/terraform/language/values/variables)
- [Terraform Local Values](https://developer.hashicorp.com/terraform/language/values/locals)
- [Terraform Output Values](https://developer.hashicorp.com/terraform/language/values/outputs)
