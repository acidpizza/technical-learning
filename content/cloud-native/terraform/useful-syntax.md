---
title: "Useful Syntax"
weight: 3
---

## Here Document

```hcl
user_data = <<-EOF
  #!/bin/bash
  echo "Hello, world" > index.html
  nohup busybox httpd -f -p 8080 &
  EOF
```

## Variable Mapping

```hcl
locals {
  # Raise error if no valid number
  vm_memory_mb = (
    var.vm_memory == 1  ? 1024  :
    var.vm_memory == 2  ? 2048  :
    "ERROR: vm_memory is not valid"
  )

  # Raise error if no valid string
  org_name = (
    var.org_name == "marvel" ? var.org_name :
    var.org_name == "dc"     ? var.org_name :
    file("ERROR: org_name is not valid")
  )
}
```

