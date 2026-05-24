
# 📘 Terraform Variables 


# 📌 1. What are Terraform Variables?

## 💡 Definition

Terraform variables are **input parameters** used to make your configuration **dynamic and reusable**.

👉 They allow you to pass values **without modifying code**.
# 📌 2. Basic Syntax

```hcl
variable "instance_type" {
  description = "EC2 instance type"
  type        = string
  default     = "t2.micro"
}
```

# 📌 3. How to Use Variables

```hcl
resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = var.instance_type
}
```

✔️ Access using:

```hcl
var.<variable_name>
```


# 📌 4. Variable Types

## 🔹 String

```hcl
type = string
```

## 🔹 Number

```hcl
type = number
```

## 🔹 Boolean

```hcl
type = bool
```

## 🔹 List

```hcl
type = list(string)
```

## 🔹 Map

```hcl
type = map(string)
```


# 📌 5. Ways to Assign Values

## 1. Default Value

```hcl
default = "t2.micro"
```

## 2. CLI (`-var`)

```bash
terraform apply -var="instance_type=t2.large"
```
## 3. `.tfvars` File (Most Common 💡)

A `.tfvars` file is used to **assign values to variables** in Terraform.

👉 It does NOT define variables — it only **provides values**.

```hcl
# terraform.tfvars
instance_type = "t2.medium"
```

## 4. Environment Variables

```bash
export TF_VAR_instance_type="t2.nano"
```
## 5. Auto-loaded tfvars files

Terraform automatically loads:
```id="v013"
terraform.tfvars
*.auto.tfvars
```
Any file named  terraform.tfvars  or ending in  .auto.tfvars  (or their  .json  variants) is read automatically by Terraform, so you don’t have to manually specify  -var-file  for them.

```
terraform apply -var-file prod.tfvars
```

# 📌 6. 🔥 Variable Priority (VERY IMPORTANT)

When the same variable is defined in multiple places, Terraform follows this priority (highest → lowest):


## 🎯 Example

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

If:
- `.tfvars` → `"t2.medium"`
- CLI → `"t2.large"`

👉 Final value = **t2.large** ✔️

---

# 📌 7. Required Variables

```hcl
variable "region" {
  type = string
}
```

❗ No default → Terraform will ask for input

# 📌 8. Variable Validation 

```hcl
variable "env" {
  type = string

  validation {
    condition     = contains(["dev", "stage", "prod"], var.env)
    error_message = "Must be dev, stage, or prod"
  }
}
```

# 📌 9. Real-World Example

```hcl
variable "env" {
  type    = string
  default = "dev"
}

variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t2.micro"
    prod = "t2.large"
  }
}

resource "aws_instance" "example" {
  ami           = "ami-12345678"
  instance_type = var.instance_types[var.env]

  tags = {
    Environment = var.env
  }
}
```


# 📌 10. Best Practices

✔️ Always define `type`  
✔️ Add `description`  
✔️ Use `.tfvars` for environments  
✔️ Use validation rules  
✔️ Avoid hardcoding values


# 📌 11. Variables vs Locals

|Feature|Variables|Locals|
|---|---|---|
|Input|✔️ external|❌ internal|
|Changeable|✔️ yes|❌ no|
|Use case|user input|computed values|



# 📌 Terraform Variable Types (Basic)

Terraform supports several types, but the core ones are:

- `string`
- `number`
- `bool` (boolean)

Terraform supports basic variable types like string, number, and bool. String is used for text values, number for numeric values, and bool for true/false conditions. These types help enforce input validation and prevent configuration errors.
# 🟢 1. `string`

## 💡 Definition

A `string` is a **sequence of characters (text)**.

## ✅ Example

```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}
```

## 📦 Usage

```hcl
resource "aws_instance" "example" {
  instance_type = var.instance_type
}
```

# 🔵 2. `number`

## 💡 Definition

A `number` represents **numeric values** (integer or decimal).

## ✅ Example

```hcl
variable "instance_count" {
  type    = number
  default = 2
}
```

## 📦 Usage

```hcl
resource "aws_instance" "example" {
  count = var.instance_count
}
```


# 🟡 3. `bool` (Boolean)

## 💡 Definition

A `bool` represents **true or false values**.


## ✅ Example

```hcl
variable "create_instance" {
  type    = bool
  default = true
}
```

## 📦 Usage

```hcl
resource "aws_instance" "example" {
  count = var.create_instance ? 1 : 0
}
```


# 📊 Quick Comparison

|Type|Example Value|Notes|
|---|---|---|
|string|`"t2.micro"`|Always in quotes|
|number|`2`|No quotes|
|bool|`true`|No quotes|


# 📦 Combined Example of string, number,bool

```hcl
variable "instance_type" {
  type    = string
  default = "t2.micro"
}

variable "instance_count" {
  type    = number
  default = 2
}

variable "create_instance" {
  type    = bool
  default = true
}
```


## Usage:

```hcl
resource "aws_instance" "example" {
  count         = var.create_instance ? var.instance_count : 0
  instance_type = var.instance_type
}
```


# 🎯 When to Use What?

- `string` → names, IDs, regions
    
- `number` → counts, sizes
    
- `bool` → enable/disable features
    


# 🎯 Interview Answer

> Terraform supports basic variable types like string, number, and bool. String is used for text values, number for numeric values, and bool for true/false conditions. These types help enforce input validation and prevent configuration errors.

---

# ✅ Confidence

**100%** — These are Terraform core fundamentals.

---

If you want next step 💡  
We can go into:

- **list, map, object (VERY important for real projects)**
    
- or **type validation tricks used in production**
    

Just tell me 👍
---

# 🎯 Interview Answer

> Terraform variables are used to parameterize configurations, allowing dynamic and reusable infrastructure. Their values can be assigned through CLI, tfvars files, environment variables, or defaults, with a defined priority order.

---

# 🚀 Pro Tip

👉 In real projects:

- Use **variables for inputs**
    
- Use **locals for logic**
    
- Combine both for clean design 💡
    

---

# ✅ Confidence

**100%** — This aligns with Terraform official behavior and real-world usage.

# 📦 Terraform Variables & `.tfvars` — Notes

---

# 🧠 1. Why Variables?

> Variables make Terraform code:

- Reusable ✔️
- Clean ✔️
- Environment-independent ✔️

# 🧱 2. Standard Project Structure (IMPORTANT)

```text
main.tf          → actual resources
variables.tf     → variable declarations
terraform.tfvars → variable values
```


# 📌 3. variables.tf (Declaration Only)

👉 Define variables (NO values required)

```hcl
variable "ami" {
  description = "AMI for EC2"
  type        = string
}
```

# 📌 4. terraform.tfvars (Values)

👉 Assign values here

```hcl
ami = "ami-0abcdef123456"
```

# 🧠 Key Rule

```text
variables.tf     → defines variable
terraform.tfvars → provides value
```


# 🔄 5. How Terraform Picks Values

Priority order:

```text
1. CLI (-var or -var-file)
2. *.tfvars file
3. default value in variable
```

## Example

```hcl
variable "ami" {
  default = "ami-default"
}
```

```hcl
ami = "ami-from-tfvars"
```

👉 Terraform uses:

```text
ami-from-tfvars ✔️
```

# ⚠️ Important Rule

> `.tfvars` overrides `default`


# 🧪 6. Default Value Behavior

## Case 1 — Only default

```hcl
variable "ami" {
  default = "ami-123"
}
```

✔️ Used automatically

---

## Case 2 — tfvars present

```hcl
ami = "ami-999"
```

✔️ tfvars value is used  
❌ default ignored


# 📁 7. Multiple Environments (VERY IMPORTANT)

## Example Files:

```text
dev.tfvars
prod.tfvars
```


## dev.tfvars

```hcl
instance_type = "t2.micro"
```


## prod.tfvars

```hcl
instance_type = "t2.large"
```

---

## Run Commands

### For Dev:

```bash
terraform plan -var-file="dev.tfvars"
```

---

### For Prod:

```bash
terraform plan -var-file="prod.tfvars"
```

---

# 🔥 Benefit

```text
Same code → different environments
```


# ⚠️ 8. File Naming Behavior

## ✅ Special Name (Auto-loaded)

```text
terraform.tfvars
```

👉 Terraform loads it automatically ✔️


## ❌ Custom Name

```text
prod.tfvars
```

👉 Must specify:

```bash
terraform plan -var-file="prod.tfvars"
```

# 🧪 9. Practical Example


## 📄 variables.tf

```hcl
variable "ami" {
  type = string
}
```


## 📄 terraform.tfvars

```hcl
ami = "ami-0c55b159cbfafe1f0"
```

---

## 📄 main.tf

```hcl
resource "aws_instance" "web" {
  ami           = var.ami
  instance_type = "t2.micro"
}
```

---

# ▶️ Run

```bash
terraform init
terraform plan
terraform apply
```

---

# 💡 10. Production Insight

👉 Best practice:

- Never hardcode values in `main.tf`
    
- Use:
    
    - `variables.tf` → structure
        
    - `.tfvars` → environment-specific values
        

---



# 🎯 Summary

- Variables improve reusability ✔️
- `.tfvars` stores values ✔️
- Supports multiple environments ✔️
- `terraform.tfvars` auto-loads ✔️
- Custom `.tfvars` requires CLI flag ✔️



# 📦 Terraform Variable Assignment

---

# 🧠 1. What if Variable Has NO Value?

```hcl
variable "instance_type" {}
```

```hcl
resource "aws_instance" "web" {
  instance_type = var.instance_type
}
```

---

## ▶️ Run

```bash
terraform plan
```

👉 Terraform will ask:

```text
Enter a value for variable "instance_type":
```

✔️ You must input manually  
❗ Needed again for `apply`

---

# 🔥 2. Ways to Assign Variable Values

---

# ✅ 1. Default Value (Inside variable)

```hcl
variable "instance_type" {
  default = "t2.micro"
}
```

✔️ Automatically used  
✔️ No prompt

---

# ✅ 2. `.tfvars` File (Most Common)

```hcl
instance_type = "t2.small"
```

Run:

```bash
terraform apply
```

✔️ Auto-loaded if file is `terraform.tfvars`

---

# ✅ 3. CLI Argument (`-var`)

```bash
terraform apply -var="instance_type=t2.large"
```

✔️ No prompt  
✔️ Overrides default & tfvars

---

# ✅ 4. Environment Variable (Linux) 🔥

## 📌 Syntax:

```bash
export TF_VAR_instance_type="t2.medium"
```

---

## ▶️ Then run:

```bash
terraform plan
```

✔️ Terraform picks value automatically

---

# ⚠️ Important Rule

```text
Prefix MUST be: TF_VAR_
```

Example:

```bash
export TF_VAR_region="us-east-1"
```

---

# ⚠️ Important (Linux Behavior)

If not working:

👉 Restart terminal OR:

```bash
source ~/.bashrc
```

---

# 🧠 3. Priority Order (VERY IMPORTANT)

```text
1. CLI (-var, -var-file)
2. tfvars file
3. Environment variables
4. Default value
5. Prompt (last fallback)
```

---

# 🧪 Example (Linux Flow)

```bash
export 

="t2.micro"
```

```bash
terraform plan
```

✔️ Uses environment variable

---

```bash
terraform plan -var="instance_type=t2.large"
```

✔️ Overrides env variable

---

# 🎯 Final Summary

- Terraform asks input if no value ❗
    
- You can assign values using:
    

|Method|Use Case|
|---|---|
|default|fallback|
|tfvars|environments|
|CLI (-var)|quick override|
|env variable|automation (CI/CD)|
