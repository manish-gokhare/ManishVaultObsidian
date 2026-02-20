 - Understand basic Terraform Commands
    - terraform init
    - terraform validate
    - terraform plan
    - terraform apply
    - terraform destroy




Terraform has a simple core workflow: write configuration, then run `init`, `validate`, `plan`, `apply`, and finally `destroy` when you want to remove everything. [1][5]

## Overall workflow

- You first write your `.tf` files that describe the desired infrastructure (resources, variables, providers). 
- Then you follow the core lifecycle: `terraform init` → `terraform validate` → `terraform plan` → `terraform apply` → `terraform destroy` when cleaning up. 

## terraform init

- `terraform init` is run once per new project (or when you change providers/modules) to initialize the working directory. It downloads provider plugins (like AWS, Azure, GCP) and sets up the backend for state if configured. 
- Without a successful `init`, the other commands will fail because Terraform does not yet know which providers and modules to use.

## terraform validate

- `terraform validate` checks that your configuration files are syntactically correct and internally consistent.
- It does not contact cloud providers or show what will change; it only catches errors such as wrong argument names, missing required attributes, or invalid expressions before you plan/apply. 

## terraform plan

- `terraform plan` creates an execution plan (a “dry run”) showing what Terraform intends to add, change, or destroy to reach the desired state. 
- You use this output to review and confirm the changes are safe and expected before actually touching real infrastructure; it can also be saved to a plan file for applying later. 
- terraform plan -o myplan

## terraform apply

- `terraform apply` takes the planned changes and performs them against the real cloud environment, creating, updating, or deleting resources to match your configuration. 
- If you don’t give it a saved plan file, it will automatically run a plan step first, show you the changes, and ask for confirmation (unless you use `-auto-approve`). 

## terraform destroy

- `terraform destroy` removes the resources that are currently tracked in the Terraform state for that configuration, effectively tearing down the infrastructure. 
- It is typically used when you no longer need an environment (like a test or demo) and want to avoid ongoing costs; for normal updates you just use `plan` and `apply` without destroying everything. 



Practicals:
/Users/manish/Documents/TerraformByKalyan/02-TF-Basics/02-02-Command-Basics