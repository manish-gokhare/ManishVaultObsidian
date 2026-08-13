
Install Terraform
```
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform version
```


To work with Terraform on AWS.

- Create the user - terraform in IAM
- Provide Full admin access
- Create the access key

Configure AWS CLI
```
aws configure
```


# Terraform Registry

https://registry.terraform.io/

Terraform command
terraform init  > It download the configured provider (.terraform and .terraform.lock.hcl)
```
manish@MacBook-Pro-va-FOX 03-Terraform-Settings % ls -la
total 32
drwxr-xr-x  7 manish  staff   224 May 25 10:47 .
drwxr-xr-x@ 6 manish  staff   192 May 24 21:14 ..
drwxr-xr-x@ 3 manish  staff    96 May 25 10:47 .terraform
-rw-r--r--@ 1 manish  staff  1407 May 25 10:47 .terraform.lock.hcl
-rw-r--r--@ 1 manish  staff  1110 May 25 00:09 app1-install.sh
-rw-r--r--@ 1 manish  staff   227 May 24 23:46 c1-version.tf
-rw-r--r--@ 1 manish  staff   223 May 25 10:47 c2-ec2-instance.tf
```


terraform validte
terraform fmt
t



