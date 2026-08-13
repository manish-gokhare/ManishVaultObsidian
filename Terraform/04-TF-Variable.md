
In Terraform, **variables** are used to make your configuration reusable and customizable. Instead of hardcoding values, you define variables and assign values at runtime.

### 1. Define a Variable

Create a variable in `variables.tf`:

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

### 2. Use the Variable

Reference it using `var.<variable_name>`:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = var.instance_type
}
```

### 3. Assign Values

#### Option A: Use Default Value

Terraform automatically uses the default:

```bash
terraform apply
```

#### Option B: Via Command Line

```bash
terraform apply -var="instance_type=t3.small"
```

#### Option C: Using a `.tfvars` File

`terraform.tfvars`

```hcl
instance_type = "t3.medium"
```

Apply:

```bash
terraform apply
```

Or specify a file:

```bash
terraform apply -var-file="dev.tfvars"
```

### 4. Variable Types

```hcl
variable "environment" {
  type = string
}

variable "instance_count" {
  type = number
}

variable "enable_monitoring" {
  type = bool
}

variable "subnets" {
  type = list(string)
}

variable "tags" {
  type = map(string)
}
```

### 5. Complex Variables

#### List

```hcl
variable "availability_zones" {
  type = list(string)
  default = ["us-east-1a", "us-east-1b"]
}
```

Usage:

```hcl
availability_zone = var.availability_zones[0]
```

#### Object

```hcl
variable "server_config" {
  type = object({
    instance_type = string
    volume_size   = number
  })
}
```

Value:

```hcl
server_config = {
  instance_type = "t3.micro"
  volume_size   = 20
}
```

Usage:

```hcl
instance_type = var.server_config.instance_type
```

### 6. Validation

```hcl
variable "environment" {
  type = string

  validation {
    condition     = contains(["dev", "test", "prod"], var.environment)
    error_message = "Environment must be dev, test, or prod."
  }
}
```

### 7. Sensitive Variables

```hcl
variable "db_password" {
  type      = string
  sensitive = true
}
```

Provide the value:

```bash
export TF_VAR_db_password="mysecretpassword"
```

### Example Project Structure

```text
terraform-project/
├── main.tf
├── variables.tf
├── terraform.tfvars
└── outputs.tf
```


## Variable Value Precedence

When the same variable is assigned in multiple places, Terraform uses the **highest-precedence source**.

From **lowest to highest precedence**:

| Precedence  | Source                                                                                      |
| ----------- | ------------------------------------------------------------------------------------------- |
| 1 (Lowest)  | Variable `default` value in `variables.tf`                                                  |
| 2           | Environment variables (`TF_VAR_name`)                                                       |
| 3           | `terraform.tfvars`                                                                          |
| 4           | `terraform.tfvars.json`                                                                     |
| 5           | `*.auto.tfvars` and `*.auto.tfvars.json` (loaded automatically, processed in lexical order) |
| 6           | `-var-file` command-line option (later files override earlier files)                        |
| 7 (Highest) | `-var` command-line option                                                                  |
