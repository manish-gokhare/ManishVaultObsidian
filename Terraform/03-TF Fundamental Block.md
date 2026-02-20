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
Terraform relies on providers to interact with Remote system.
Terraform install provider and use them to interact with provider resources.
Provider configuration belongs to root modules.

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