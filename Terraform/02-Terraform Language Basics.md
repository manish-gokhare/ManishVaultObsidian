
Understand about Arguments, Attributes and Meta-Arguments.

In Terraform, “arguments” set inputs on a resource, “attributes” are the values that resource exposes (often outputs), and “meta-arguments” are special arguments that change how Terraform manages the resource (count, for_each, lifecycle, etc.). 

**Arguments**
Arguments are the normal configuration settings you write inside a resource, data, module, or provider block.  Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"    # argument
  instance_type = "t3.micro"       # argument
  tags = {                         # argument
    Name = "web-server"
  }
}

```
Here `ami`, `instance_type`, and `tags` are arguments you provide to configure the EC2 instance.

**Attributes**
Attributes are the readable values of a resource: some you set (from arguments), others Terraform or the provider fill in after creation

```hcl
output "web_public_ip" {
  value = aws_instance.web.public_ip
}
```
`public_ip` is an attribute of `aws_instance.web` that is known only after the instance is created and then can be used in expressions or outputs.

Meta-Arguments

Meta-arguments are special arguments that work with almost all resources/modules and control behavior like how many to create, dependencies, provider selection, and lifecycle rules.  Main built‑in meta-arguments:
	•	`count`
	•	`for_each`
	•	`depends_on`
	•	`provider` / `providers`
	•	`lifecycle


**Terraform top Level Block**

- Terraform Settings Block 
- Provider Block
- Resource Block
- Input Variables Block
- Output Values Block
- Local Values Block
- Data Sources Block
- Modules Block


Terraform configuration files are made of several top-level blocks, each with a specific purpose, and together they describe how Terraform should behave and what infrastructure to manage. [5]

## Terraform settings block

The `terraform` block configures Terraform itself: CLI version constraints, required providers, and optional backend or cloud settings. 
Example:

```hcl
terraform {
  required_version = ">= 1.7.0"

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}
```

This block does not create resources; it controls how Terraform runs and which providers and versions are allowed. 

## Provider block

A `provider` block configures connection details for a specific cloud or service, such as region, credentials, or endpoints. 
Example:

```hcl
provider "aws" {
  region = "us-east-1"
}
```

Every resource that uses `aws` will, by default, use this configuration unless you override it with an alias. [5]

## Resource block

A `resource` block describes real infrastructure objects that Terraform will create, update, or delete, such as VMs, networks, or databases. 
Example:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t3.micro"
}
```

The first label (`aws_instance`) is the resource type, and the second (`web`) is the local name used to reference this instance in other parts of the configuration. 

## Input variables block

A `variable` block defines an input value that can be passed from the user, CLI, or environment, allowing reuse and parameterization. 
Example:

```hcl
variable "instance_type" {
  description = "EC2 size for web server"
  type        = string
  default     = "t3.micro"
}
```

You then use it as `var.instance_type` inside other blocks, which makes the configuration flexible between environments. 

## Output values block

An `output` block exposes useful values after `terraform apply`, such as IP addresses or IDs, which can be read from CLI or used by other tooling.
Example:

```hcl
output "web_public_ip" {
  value = aws_instance.web.public_ip
}
```

Outputs are often used to quickly find important details after deployment or to pass data between modules or systems. 

## Local values block

A `locals` block defines local names for expressions or derived values, helping avoid repetition and improving readability. 
Example:

```hcl
locals {
  common_tags = {
    project = "demo"
    owner   = "team-a"
  }
}
```

You then reuse `local.common_tags` in multiple resources instead of repeating the same tag map everywhere. 

## Data sources block

A `data` block reads existing information from providers without creating new resources, such as an existing AMI, VPC, or secret. 
Example:

```hcl
data "aws_ami" "latest" {
  most_recent = true
  owners      = ["amazon"]

  filter {
    name   = "name"
    values = ["amzn2-ami-hvm-*-x86_64-gp2"]
  }
}
```

Later, you can use `data.aws_ami.latest.id` to reference that existing image in your resources. [5]

## Modules block

A `module` block calls a reusable set of Terraform configuration (a module), which can be local or remote (e.g., Git, Terraform Registry). [5]
Example:

```hcl
module "network" {
  source = "./modules/network"

  vpc_cidr = "10.0.0.0/16"
}
```

Modules help you structure large configurations, share patterns across projects, and keep code DRY. [7]

Sources
[1] Terraform Dynamic Blocks: DRY Principle & Examples https://spacelift.io/blog/terraform-dynamic-blocks
[2] Dynamic Blocks - Configuration Language | Terraform https://developer.hashicorp.com/terraform/language/expressions/dynamic-blocks
[3] Understanding Terraform Dynamic Blocks with Examples https://kodekloud.com/blog/terraform-dynamic-block/
[4] Understanding the Terraform Block: The Configuration Foundation https://scalr.com/learning-center/understanding-the-terraform-block-the-configuration-foundation/
[5] Overview - Configuration Language | Terraform https://developer.hashicorp.com/terraform/language
[6] Style Guide - Configuration Language | Terraform https://developer.hashicorp.com/terraform/language/style
[7] Structuring Terraform and OpenTofu https://scalr.com/guides/platform-engineers-guide-to-structuring-terraform-and-opentofu
[8] Best practices for code base structure and organization https://docs.aws.amazon.com/prescriptive-guidance/latest/terraform-aws-provider-best-practices/structure.html
[9] Organizing Terraform Code for Scalability and Maintainability https://terrateam.io/blog/terraform-code-organization
[10] Structure of Terraform Configuration Language and its Syntax https://www.linkedin.com/pulse/terraform-easy-2-structure-configuration-language-its-srivastava-9wwvc
