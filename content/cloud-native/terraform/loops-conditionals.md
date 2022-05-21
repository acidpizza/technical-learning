---
title: "Loops and Conditionals"
weight: 2
---

## Loops

https://blog.gruntwork.io/terraform-tips-tricks-loops-if-statements-and-gotchas-f739bbae55f9

### count

```hcl
variable "user_names" {
  description = "Create IAM users with theses names"
  type        = list(string)
  default     = ["neo", "trinity", "morpheus"]
}

# Create 3 users from list
resource "aws_iam_user" "example" {
  count = length(var.user_names)
  name = var.user_names[count.index]
}

# Create 3 users with running number
resource "aws_iam_user" "example2" {
  count = 3
  name = "user-${count.index}"
}
```

**Limitations of count:**

1. Cannot use count to loop over inline blocks within a resource

```hcl
# Cannot use count to dynamically create inline blocks
resource "x" "y" {
  inline_block_1 {
    ...
  }
  inline_block_2 {
    ...
  }
}
```

2. Changing elements in the middle of the list will affect the position of other elements, causing a **recreate of all the subsequent elements**.


### for_each

> Use for_each instead of count as far as possible

for_each can create **multiple inline blocks**.

```hcl
resource "aws_autoscaling_group" "example" {
  dynamic "tag" {
    for_each = var.custom_tags

    content {
      key                 = tag.key
      value               = tag.value
      propagate_at_launch = true
    }
  }
}
```

for_each only accepts set or map and hence **allows removal of elements from middle of collection safely**.

```hcl
resource "aws_iam_user" "example" {
  for_each = toset(var.user_names)
  name     = each.value
}
```

**for**

Loop over a collection to **generate a single value**.

```hcl
locals {
  names = ["neo", "trinity", "morpheus"]
}

output "upper_names" {
  value = [for name in local.names : upper(name) if length(name) < 5]
}
```

```hcl
variable "hero_thousand_faces" {
  description = "map"
  type        = map(string)
  default     = {
    neo      = "hero"
    trinity  = "love interest"
    morpheus = "mentor"
  }
}
output "bios" {
  value = [for name, role in var.hero_thousand_faces : "${name} is the ${role}"]
}

output "upper_roles" {
  value = {for name, role in var.hero_thousand_faces : upper(name) => upper(role)}
}
```


## Conditionals

### Create resources conditionally

```hcl
variable "my_flag" {
  description = "Boolean value"
  type        = bool
}

resource "resource_one" "foo" {
  count = var.my_flag ? 1 : 0
  ...
}

resource "resource_two" "bar" {
  count = var.my_flag ? 0 : 1
  ...
}
```

### Create inline blocks conditionally

```hcl
dynamic "tag" {
  for_each = {
    for key, value in var.custom_tags:
      key => upper(value)
      if key != "Name"
  }

  content {
    key                 = tag.key
    value               = tag.value
    propagate_at_launch = true
  }
}

```

## Gotchas

1. `count` and `for_each` cannot accept computed resource outputs. It can only accept hard-coded values.

```hcl
resource "aws_instance" "example_3" {
  count         = random_integer.num_instances.result
  ami           = "ami-0c55b159cbfafe1f0"
  instance_type = "t2.micro"
}
```
```text
Error: Invalid count argument
on main.tf line 30, in resource "aws_instance" "example_3":
  30:   count         = random_integer.num_instances.result
The "count" value depends on resource attributes that cannot be determined until apply, so Terraform cannot predict how many instances will be created. To work around this, use the -target argument to first apply only the resources that the count depends on.
```
