---
title: "Open Policy Agent"
weight: 1
---

## References

- https://medium.com/@mathurvarun98/how-to-write-great-rego-policies-dc6117679c9f


## Overview

- Decouples decision making from policy enforcement.
- JSON input and output
- Rego: language used to express policies
- File Types
  - Input: JSON data to evaluate policies on
  - Data: Rego packages to implement policy 


## OPA CLI

```bash
# Test policy
opa eval <command>
opa eval -i input.json -d mypackage.rego 'data.mypackage.myrule'

# Interactive shell
opa run 
exit or ctrl-d

# Run as server
opa run --server ./mypackage.rego # default runs on 0.0.0.0:8181
curl -v localhost:8181/v1/data/mypackage/myrule -d @input.json -H 'Content-Type: application/json'
```

## Rego

### Basics

```bash
# Assign variables (:=)
myvar := "1"
rect := { "width": 2, "height": 4 }

# Cannot reassign variable
myvar := x
myvar := y # Error

# Comparison (==)
rect == { "height": 4, "width": 2 }

# Results
# True     -> true
# Not true -> "undefined decision"
true

# Unification (=): assign to variable if expression is true
# eg. x = y
# if both x and y have values, it acts as ==
# if either does not, it acts as :=
# if neither has a value, it throws an error
# Using = is a declarative programming style
```

### Arrays and Objects

```bash
# { 
#   "servers": [ { "id": "1", "protocols": [ "A", "B" ] }, … ] 
# }
# Query is bound to variable 'input'
# Values are referenced with dot operator
input.servers[0].protocols[0]
input["servers"][0]["protocols"][0]

# count
count(input.servers) >= 1

# lookup last value
val := arr[count(arr)-1]

# check if "foo" does not belong to the set
not a_set["foo"]

# iterate over indices i
arr[i]

# iterate over values
val := arr[_]

# iterate over index-value pairs
val := arr[i]

# iterate over keys
obj[key]

# iterate over values
val := obj[_]

# iterate over key-value pairs
val := obj[key]

# -----------------------

# find key k whose bar.baz array index i is 7
foo[k].bar.baz[i] == 7

# simultaneous: find keys in objects foo and bar with same value
foo[k1] == bar[k2]

# simultaneous self: find 2 keys in object foo with same value
foo[k1] == foo[k2]; k1 != k2

# multiple conditions: k has same value in both conditions
foo[k].bar.baz[i] == 7; foo[k].qux > 3

# -----------------------

# assert no values in set match predicate
count({x | set[x]; f(x)}) == 0

# assert all values in set make function f true
count({x | set[x]; f(x)}) == count(set)

# assert no values in set make function f true (using negation and helper rule)
not any_match

# assert all values in set make function f true (using negation and helper rule)
not any_not_match
```

### Rules

```bash
# Rule collapsed form
a { x := 42; y := 41; x > y }

# Rule long form
# a is true if ...
a = true {
  x > y   # Order of operations does not matter
  x := 41
  y := 42
}
a {       # equivalent definition. =true is implicit.
  x > y
  x := 41
  y := 42
}

# Conditionals
default a = 1
a = 5 { ... }
a = 100 { ... }

# Else
authorize := "allow" {
  input.user == "superuser" # Allow superuser to perform any operation
}
else := "deny" {
  input.path[0] == "admin"           # disallow 'admin operations'
  input.source_network == "external" # from external networks
}
else := "allow"

# Incremental
# a_set will contain values of x and values of y
a_set[x] { ... }
a_set[y] { ... }
# a_map will contain key->value pairs x->y and w->z
a_map[x] = y { ... }
a_map[w] = z { ... }

# Logic
# AND operator = ;
a == "1"; b == "2"
# Alternatively, AND can be split in multiple lines
a == "1"
b == "2"

# OR operator
# -> use multiple rules with same name
shell_executable := true {
  input.protocol == "telnet"
}
shell_executable := true {
  input.protocols == "ssh"
}
shell_executable
# -> Use incremental rules to add on to result set
# {
#   "servers": [
#     {
#       "id": "busybox",
#       "protocols": ["http", "telnet"]
#     },
#     {
#       "id": "db",
#       "protocols": ["mysql", "ssh"]
#     },
#     {
#       "id": "web",
#       "protocols": ["https"]
#     }
#   ]
# }
shell_accessible[server.id] {
    server := input.servers[_]
    server.protocols[_] == "telnet"
}
shell_accessible[server.id] {
    server := input.servers[_]
    server.protocols[_] == "ssh"
}
shell_accessible
# [
#   "busybox",
#   "db"
# ]
```

### Functions

```bash
trim_and_split(s) := x {
  t := trim(s, " ")
  x := split(t, ".")
}
```
```bash
trim_and_split("   foo.bar ")
# result
# [
#   "foo",
#   "bar"
]
```

### Loop

```bash
# -> use variable to substitute array index
some i; input.networks[i].public == true        # i now contains true indexes
some i, j; input.servers[i].protocols[j] == "A"

# -> Can also list over multiple lines. Variables will match all constraints.
some i, j
id := input.ports[i].id
input.ports[i].network == input.networks[j].id
input.networks[j].public

# Wildcard variable
# -> Every instance is a unique instance
# -> Are there any server protocols == A?
input.servers[_].protocols[_] == "A"
```

### Modules

Policies are defined inside modules. A module consists of:
- 1 package declaration
- 0 or more import statements
- 0 or more rule definitions

```bash
# Define package
package mypackage.constants
pi := 3.14

# Import package
import data.servers
```