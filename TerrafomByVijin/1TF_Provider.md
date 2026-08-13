
#  # Terraform Providers — Notes

## What is a Provider in Terraform?

A **provider** is a plugin that allows Terraform to interact with external platforms or services.

Examples:

- AWS
- Azure
- Google Cloud
- Kubernetes
- Docker
- Local system

Terraform itself does not know how to create resources.  
Providers add that capability.

Example:

- AWS provider → creates EC2, S3, VPC
- Local provider → creates local files
- Docker provider → manages containers


# Terraform Workflow with Providers

After writing Terraform configuration files:

## Step 1 — Run Initialization

```bash
terraform init
```

This command:

- Initializes the Terraform working directory
- Downloads provider plugins
- Installs required providers
- Prepares Terraform to run

# Important Point

```bash
terraform init
```

is a **safe command**.

You can run it multiple times without affecting infrastructure.

It only prepares the environment.

# Provider Plugin Architecture

Terraform uses a:

> Plugin-based architecture

Meaning:

- Every provider is a separate plugin
- Terraform dynamically downloads only required providers
    

This allows Terraform to support:

- Hundreds of cloud platforms
- SaaS products
- Networking tools
- Databases
- Local resources

# Where Providers are Stored

After `terraform init`, plugins are downloaded into:

```text
.terraform/
```

Inside the working directory.


# Terraform Registry

Providers are available in:

[Terraform Registry](https://registry.terraform.io/?utm_source=chatgpt.com)

This is the public provider repository maintained by [HashiCorp](https://www.hashicorp.com/?utm_source=chatgpt.com).


# Types (Tiers) of Providers

## 1. Official Providers

Maintained by HashiCorp.

Examples:

- AWS
    
- AzureRM
    
- Google
    
- Local
    

These are most trusted and widely used.

Examples:

- `hashicorp/aws`
    
- `hashicorp/azurerm`
    
- `hashicorp/google`
    
- `hashicorp/local`
    

---

## 2. Partner Providers

Maintained by third-party companies partnered with HashiCorp.

Examples:

- F5 BIG-IP
    
- Heroku
    
- DigitalOcean
    

These companies officially support the providers.

---

## 3. Community Providers

Maintained by individual contributors from the Terraform community.

Not officially supported by HashiCorp.

Use carefully in production.

---

# Provider Source Address

Example:

```text
hashicorp/local
```

This is called the:

> Source Address

Terraform uses it to locate and download the provider.

---

# Source Address Format

General format:

```text
[hostname]/[namespace]/[provider]
```

Example:

```text
registry.terraform.io/hashicorp/local
```

Breakdown:

|Part|Meaning|
|---|---|
|`registry.terraform.io`|Registry hostname|
|`hashicorp`|Organization namespace|
|`local`|Provider type/name|

---

# Hostname

Optional.

If omitted:

```text
hashicorp/local
```

Terraform automatically assumes:

```text
registry.terraform.io
```

So these are equivalent:

```text
hashicorp/local
```

and

```text
registry.terraform.io/hashicorp/local
```

---

# Namespace

Namespace identifies:

- Organization
    
- Company
    
- Maintainer
    

Examples:

- `hashicorp`
    
- `digitalocean`
    
- `kreuzwerker`
    

---

# Provider Type

Represents actual provider.

Examples:

- `aws`
    
- `google`
    
- `local`
    
- `random`
    

---

# Example Provider Block

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "2.0.0"
    }
  }
}
```

---

# What Happens During `terraform init`

Terraform:

1. Reads configuration files
2. Identifies required providers
3. Connects to Terraform Registry
4. Downloads provider plugins
5. Stores plugins locally

Example output:

```text
Installing hashicorp/local v2.0.0...
```

---

# Default Provider Version Behavior

By default:

> Terraform installs the latest provider version

This can sometimes cause problems because:

- New versions may introduce breaking changes
    
- Existing code may stop working
    

---

# Provider Version Locking

Best practice:

- Lock provider versions
    

Example:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

Meaning:

- Allow updates within 5.x
    
- Prevent upgrade to 6.x
    

---

# Why Version Locking is Important

Without locking:

- Team members may use different versions
    
- CI/CD may behave differently
    
- Infrastructure changes may become unpredictable
    

---

# Real-World Example

## Configuration

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "2.0.0"
    }
  }
}

resource "local_file" "pet" {
  filename = "${path.module}/pets.txt"
  content  = "Terraform Providers"
}
```

---

# Relation with `path.module`

Here:

```hcl
"${path.module}/pets.txt"
```

means:

- Create `pets.txt`
- Inside current module directory
This ensures file operations work correctly regardless of where Terraform is executed from.

---

# Quick Revision Notes

## Provider

Plugin used by Terraform to interact with platforms.

---

## terraform init

Initializes directory and downloads providers.

---

## Terraform Registry

Public location where providers are stored.

---

## Source Address Format

```text
[hostname]/[namespace]/[provider]
```

---

## Provider Types

1. Official
    
2. Partner
    
3. Community
    

---

## Default Registry

```text
registry.terraform.io
```

---

## Best Practice

Always lock provider versions.

# Interview Questions

## Q1. What is a Terraform provider?

A provider is a plugin that enables Terraform to manage resources on a platform or service.


## Q2. What does `terraform init` do?

It initializes the working directory and downloads required provider plugins.


## Q3. Where are providers downloaded?

Inside the hidden `.terraform` directory.

## Q4. What is a source address?

Unique identifier Terraform uses to locate provider plugins.

Example:

```text
hashicorp/aws
```


## Q5. Why should provider versions be locked?

To avoid breaking changes and ensure consistent infrastructure deployments.


# Terraform Configuration Files — Notes

## What is a Terraform Configuration File?

A Terraform configuration file contains:

- Resource definitions
- Provider configuration
- Variables
- Outputs
- Infrastructure settings

Terraform configuration files use:

```text
.tf
```

extension.

Example:

```text
main.tf
providers.tf
variables.tf
outputs.tf
```


# Configuration Directory

Terraform works inside a:

> Configuration Directory

This is the folder containing `.tf` files.

Example:

```text
terraform-local-file/
```

Inside:

```text
main.tf
cat.tf
variables.tf
```


# Important Rule

Terraform automatically reads:

> ALL `.tf` files in the configuration directory

You do NOT need to explicitly import them.


# Example

## local.tf

```hcl
resource "local_file" "dog" {
  filename = "${path.module}/dog.txt"
  content  = "Dog file"
}
```


## cat.tf

```hcl
resource "local_file" "cat" {
  filename = "${path.module}/cat.txt"
  content  = "Cat file"
}
```

---

When you run:

```bash
terraform apply
```

Terraform processes BOTH files together.

Result:

- `dog.txt` created
    
- `cat.txt` created
    

---

# Key Concept

Terraform treats all `.tf` files in the directory as:

> One combined configuration

Terraform does NOT execute files individually.


# File Naming Conventions

Terraform does not require special names.

But in real projects, standard naming conventions are followed for readability and organization.

---

# Common Terraform Files

## 1. main.tf

Usually contains:

- Main resources
    
- Core infrastructure
    

Example:

- EC2 instances
    
- VPCs
    
- Storage
    

This is the primary configuration file.

---

## 2. providers.tf

Contains:

- Provider configuration
    
- Provider versions
    

Example:

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region = "us-east-1"
}
```

---

# 3. variables.tf

Contains:

- Input variable declarations
    

Example:

```hcl
variable "instance_type" {
  type = string
}
```

Used to make configurations reusable.

---

# 4. outputs.tf

Contains:

- Output values displayed after deployment
    

Example:

```hcl
output "instance_id" {
  value = aws_instance.web.id
}
```

---

# 5. terraform.tfvars

Contains:

- Actual values for variables

Example:

```hcl
instance_type = "t2.micro"
```

---

# Typical Terraform Project Structure

```text
project/
│
├── main.tf
├── providers.tf
├── variables.tf
├── outputs.tf
├── terraform.tfvars
│
└── modules/
```

---

# Single File vs Multiple Files

## Single File Approach

Everything inside:

```text
main.tf
```

Good for:

- Small projects
    
- Learning
    
- Testing
    

---

## Multiple File Approach

Separate files:

- Easier maintenance
    
- Better readability
    
- Team collaboration
    

Used in:

- Real-world production projects
    

---

# Important Interview Concept

Terraform combines all `.tf` files in a directory into:

> A single configuration

Order of file names does NOT matter.

Example:

```text
a.tf
z.tf
main.tf
random.tf
```

Terraform loads all of them together.

---

# Relation with `path.module`

Example:

```hcl
filename = "${path.module}/cat.txt"
```

Meaning:

- Create file inside current module directory
    

This avoids path-related issues.

---

# Best Practices

## Use Standard File Names

Recommended:

- `main.tf`
    
- `providers.tf`
    
- `variables.tf`
    
- `outputs.tf`
    

Improves readability.

---

## Keep Files Modular

Do not place everything in one huge file.

Split logically.

---

## Use Comments

Example:

```hcl
# Create local file
resource "local_file" "cat" {
}
```

---

## Separate Environment Configurations

Example:

```text
dev/
prod/
stage/
```

---

# Example Complete Configuration

## providers.tf

```hcl
terraform {
  required_providers {
    local = {
      source  = "hashicorp/local"
      version = "2.0.0"
    }
  }
}
```

---

## main.tf

```hcl
resource "local_file" "dog" {
  filename = "${path.module}/dog.txt"
  content  = "Dog"
}
```

---

## cat.tf

```hcl
resource "local_file" "cat" {
  filename = "${path.module}/cat.txt"
  content  = "Cat"
}
```

---

# Commands

## Initialize

```bash
terraform init
```

---

## Preview Changes

```bash
terraform plan
```

---

## Apply Changes

```bash
terraform apply
```

---

# Quick Revision Notes

## Configuration Directory

Folder containing Terraform `.tf` files.

---

## Terraform Reads

All `.tf` files automatically.

---

## Common Files

|File|Purpose|
|---|---|
|`main.tf`|Main resources|
|`providers.tf`|Provider settings|
|`variables.tf`|Input variables|
|`outputs.tf`|Output values|
|`terraform.tfvars`|Variable values|

---

## Important Point

Terraform merges all `.tf` files into one configuration.

---

# Interview Questions

## Q1. What files does Terraform read?

Terraform reads all files with `.tf` extension in the configuration directory.

---

## Q2. Is `main.tf` mandatory?

No. It is only a naming convention.

---

## Q3. What is the purpose of `variables.tf`?

To declare reusable input variables.

---

## Q4. What is stored in `outputs.tf`?

Values Terraform displays after deployment.

---

## Q5. Does file order matter in Terraform?

No. Terraform combines all `.tf` files together automatically.


### Configuration file order of evaluation:

Terraform does **not evaluate files based on filename order** like:

```text
variables.tf → resource.tf → outputs.tf
```

Instead:

> Terraform loads ALL `.tf` files together and builds a dependency graph internally.

So file names and file order generally do **not matter**.


# Important Concept

Terraform works in these stages:

1. Load all `.tf` files
2. Parse configuration
3. Resolve variables
4. Build dependency graph
5. Plan execution
6. Apply resources in dependency order


# Your Example

Suppose you have:

## variables.tf

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```


## resource.tf

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123456"
  instance_type = var.instance_type
}
```

Terraform behavior:

- It reads BOTH files together
    
- Registers all variables
    
- Registers all resources
    
- Resolves references like:
    

```hcl
var.instance_type
```

- Then creates execution plan
    

Even if:

- `resource.tf` is read first physically
    
- `variables.tf` comes later alphabetically
    

Terraform still works.

---

# Key Point

Terraform is:

> Declarative, not procedural

Meaning:

- You describe desired state
    
- Terraform determines execution order automatically
    

Unlike scripting languages:

- Bash
    
- Python
    
- Shell scripts
    

where execution is top-to-bottom.

---

# How Terraform Determines Order

Terraform uses:

> Dependency Graph (DAG — Directed Acyclic Graph)

Dependencies are inferred from references.

Example:

```hcl
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}

resource "aws_subnet" "subnet1" {
  vpc_id = aws_vpc.main.id
}
```

Terraform understands:

```text
Subnet depends on VPC
```

So execution order becomes:

```text
1. Create VPC
2. Create Subnet
```

Even if subnet block appears before VPC block.

---

# Variable Evaluation

Variables are resolved before resource creation.

Terraform checks values in this priority order:

1. CLI flags

```bash
-var="instance_type=t3.micro"
```

2. `terraform.tfvars`
3. `.auto.tfvars`
4. Environment variables
5. Default values in variable block


# Example Full Flow

## variables.tf

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

---

## terraform.tfvars

```hcl
instance_type = "t3.micro"
```

---

## resource.tf

```hcl
resource "aws_instance" "web" {
  ami           = "ami-123"
  instance_type = var.instance_type
}
```

Terraform flow:

```text
1. Load all files
2. Read variable definitions
3. Read tfvars values
4. Resolve var.instance_type = "t3.micro"
5. Build execution graph
6. Create EC2
```

---

# What Actually Matters in Terraform

## 1. References

```hcl
aws_vpc.main.id
```

create dependencies.

---

## 2. Explicit Dependency

Sometimes you force order using:

```hcl
depends_on
```

Example:

```hcl
resource "aws_instance" "web" {
  depends_on = [aws_s3_bucket.logs]
}
```

---

# What Does NOT Matter

These generally do NOT matter:

- File names
    
- File order
    
- Resource block order
    
- Variable block order
    

---

# Mental Model

Think of Terraform as:

> "Collect everything first → then figure out dependencies"

NOT:

> "Execute line by line"

---

# Important Interview Answer

## Q: Does Terraform execute files sequentially?

No.

Terraform loads all `.tf` files together and builds a dependency graph to determine evaluation and execution order.

---

# Advanced Evaluation Order (Internal)

Terraform roughly processes in this sequence:

```text
1. Load provider configs
2. Load variables
3. Load locals
4. Build dependency graph
5. Refresh state
6. Generate plan
7. Apply resources in dependency order
```

---

# Example with Multiple Files

```text
project/
├── provider.tf
├── variables.tf
├── network.tf
├── ec2.tf
└── outputs.tf
```

Terraform treats all as:

```text
One combined configuration
```

Then resolves dependencies automatically.

---

# Common Beginner Misconception

❌ Wrong thinking:

```text
Terraform executes main.tf first
```

✅ Correct thinking:

```text
Terraform parses all files together first
```

---

# Real-World Benefit

This design allows:

- Modular configurations
    
- Team collaboration
    
- Large infrastructures
    
- Parallel resource creation
    

Terraform can even create independent resources simultaneously when no dependency exists.