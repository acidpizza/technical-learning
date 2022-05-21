---
title: "Overview"
weight: 1
---

## Resources

- https://blog.gruntwork.io/why-we-use-terraform-and-not-chef-puppet-ansible-saltstack-or-cloudformation-7989dad2865c


## Concepts

- Language: **HCL** (Hashicorp Configuration Language)
- Files
  - main.tf
  - variables.tf
  - output.tf
  - terraform.tfstate
- Pass variables
  - `TF_VAR_<name>` environment variables
  - terraform.tfvars (secret data values)
  - terraform apply -var "myvariable=value"
  - interactive prompt on terraform apply
- Use variable
  - `var.<variable name>`
  - `"prefix-${var.<variable_name>}-suffix"`


## Commands

- terraform init
- terraform plan
- terraform apply
- terraform ouput [output_name]
- terraform destroy


## Best Practices

1. Always use the **plan** command.

2. **Create before destroy:** Either use `create_before_destroy lifecycle or manually apply with new resource and then apply again without the old resource.

3. **Changing identifiers require changing state:** To prevent recreating resources, update terraform states with `terraform state mv`. Never update terraform state files by hand. Some parameters are immutable, so RTFM.