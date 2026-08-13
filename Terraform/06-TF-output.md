
### What are Outputs?

**Outputs** allow you to expose values from your Terraform configuration so they can be accessed:

- After `terraform apply`
- From the Terraform CLI (`terraform output`)
- By other modules
- In CI/CD pipelines
- From remote state backends

## Terraform Outputs — Summary Notes

### What are Outputs?

**Outputs** allow you to expose values from your Terraform configuration so they can be accessed:

- After `terraform apply`
    
- From the Terraform CLI (`terraform output`)
    
- By other modules
    
- In CI/CD pipelines
    
- From remote state backends
    

Think of outputs as the **return values of a Terraform module**.

---

## Defining an Output

Conventionally, outputs are placed in an `outputs.tf` file.

```hcl
output "s3_bucket_name" {
  value = aws_s3_bucket.project_bucket.bucket
}
```

Here:

- `s3_bucket_name` → output name
    
- `value` → resource attribute to expose
    

---

## Accessing Resource Attributes

A resource has many attributes.

Example:

```hcl
aws_s3_bucket.project_bucket.bucket
```

Where:

```text
resource_type.resource_name.attribute
```

Terraform allows outputting any available attribute.

---

## Applying Configuration

```bash
terraform apply -auto-approve
```

Terraform displays outputs at the end:

```text
Outputs:

s3_bucket_name = "demo-bucket-123"
```

---

## Retrieve Outputs from CLI

### List all outputs

```bash
terraform output
```

Example:

```text
s3_bucket_name = "demo-bucket-123"
```

---

### Retrieve a specific output

```bash
terraform output s3_bucket_name
```

Result:

```text
"demo-bucket-123"
```

---

## JSON Output Format

```bash
terraform output -json
```

Example:

```json
{
  "s3_bucket_name": {
    "sensitive": false,
    "type": "string",
    "value": "demo-bucket-123"
  }
}
```

Useful for:

- Scripts
- Automation
- CI/CD pipelines
- Parsing with tools like `jq`


## Raw Output

Without `-raw`:

```bash
terraform output s3_bucket_name
```

Output:

```text
"demo-bucket-123"
```

With `-raw`:

```bash
terraform output -raw s3_bucket_name
```

Output:

```text
demo-bucket-123
```

### Why use `-raw`?

Useful in shell scripts:

```bash
BUCKET=$(terraform output -raw s3_bucket_name)
```

Avoids problems caused by surrounding quotes.


## Output Types

Outputs are not limited to strings.

You can output:

### String

```hcl
output "bucket_name" {
  value = aws_s3_bucket.project_bucket.bucket
}
```

### Number

```hcl
output "instance_count" {
  value = 3
}
```

### List

```hcl
output "subnets" {
  value = aws_subnet.private[*].id
}
```

### Map

```hcl
output "tags" {
  value = local.common_tags
}
```

### Entire Object

```hcl
output "bucket" {
  value = aws_s3_bucket.project_bucket
}
```


## Outputs and Modules

Outputs are heavily used with modules.

### Child Module

```hcl
output "vpc_id" {
  value = aws_vpc.main.id
}
```

### Root Module

```hcl
module "network" {
  source = "./modules/network"
}

resource "aws_subnet" "private" {
  vpc_id = module.network.vpc_id
}
```

Outputs allow one module to pass information to another.

---

## Outputs and State Files

Outputs are stored in Terraform state.

Local backend:

```text
terraform.tfstate
```

Command:

```bash
terraform output
```

reads from:

```text
terraform.tfstate
```

---

## Remote Backend Example

If state is stored in S3:

```hcl
terraform {
  backend "s3" {
    bucket = "terraform-state"
    key    = "prod/terraform.tfstate"
    region = "us-east-1"
  }
}
```

Then:

```bash
terraform output
```

reads values from the remote state.

### Benefit

Different machines can access the same outputs.

Useful for:

- CI/CD pipelines
- Team collaboration
    
- Automation
    

---

## Description Attribute

You can document outputs:

```hcl
output "s3_bucket_name" {
  description = "The name of the S3 bucket"
  value       = aws_s3_bucket.project_bucket.bucket
}
```

The description is for documentation purposes and is **not shown** in:

```bash
terraform output
```

or

```bash
terraform output -json
```

---

## Sensitive Outputs

```hcl
output "db_password" {
  value     = aws_db_instance.db.password
  sensitive = true
}
```

Terraform marks it as sensitive and hides it from normal output displays.

---

## Common CI/CD Use Case

```bash
BUCKET=$(terraform output -raw s3_bucket_name)

aws s3 cp app.zip s3://$BUCKET
```

Terraform creates infrastructure and later pipeline stages consume the output values.

---

## Interview Questions

### Q1. What is an output in Terraform?

A mechanism to expose resource values from Terraform so they can be used by users, modules, scripts, or CI/CD pipelines.

---

### Q2. Where are outputs usually defined?

```text
outputs.tf
```

---

### Q3. How do you retrieve an output?

```bash
terraform output <output_name>
```

Example:

```bash
terraform output s3_bucket_name
```

---

### Q4. What is the difference between `terraform output` and `terraform output -raw`?

|Command|Result|
|---|---|
|`terraform output name`|Returns value with quotes|
|`terraform output -raw name`|Returns plain value without quotes|

---

### Q5. Can outputs be used between modules?

Yes. Outputs from one module can be referenced by another module through `module.<module_name>.<output_name>`.

---

## One-Line Memory Trick

**Variables = Inputs → Resources → Outputs = Results**

```text
User Input (var.*)
        ↓
Terraform Resources
        ↓
Output Values (output.*)
```

Outputs are essentially Terraform's way of exposing the final values created by your infrastructure.


Output command:

terraform output -json
terraform output <variable-name>

terraform output -raw s3_bkt_name > retrives without double quote.
