# Terraform Associate 004 Notes

## Taking these notes here for quick remembering.

- The prefix string `TF_VAR_` is necessary when assigning a value to a Terraform input variable through an environment variable. This prefix indicates that the environment variable is intended to set a Terraform variable. Syntax in usage is `TF_VAR_variableName`


- **Objective 4d - Understand and use complex types**

A `map(string)` is the ideal type for storing key-value pairs where you need to look up values by their keys. A list would require numeric indexing and wouldn't provide meaningful key-based lookups. A set doesn't support key-value relationships. An object would be overly complex for this simple key-value mapping and would require a predefined structure.


- When a new provider is added to the configuration, Terraform needs to download the necessary plugin for that provider. Running `terraform init` is the first step to initialize the working directory and download the required plugins, including the new provider plugin.


- **Objective 2b - Describe how Terraform uses providers**

Terraform relies on plugins called "providers" to interact with remote systems. Terraform configurations must declare which providers they require, so that Terraform can install and use them.

- Terraform builds its dependency graph automatically from attribute references between resources — these are implicit dependencies. When a resource depends on another in a way the configuration can't express through a reference, you declare it explicitly with `depends_on`, which adds the same graph edge by hand.


- Using the `-reconfigure` flag with the `terraform init` command allows you to reconfigure the backend without copying the existing state. This is the correct flag to use when changing the backend configuration without transferring the existing state data.

- A resource was changed manually outside of Terraform. You don't want to make any changes yet, but you want to see how the state would be updated to match the current real-world values. You use `terraform plan -refresh-only` to show you how the state would be updated to match the current real-world values without making any changes to the infrastructure

- A module's internal resource addresses are not visible from outside it. `module.network.aws_vpc.this.id` is not a valid reference — the parent can only see what the child explicitly declares in an output block.

- An `alias` meta-argument is used when using the same provider with different configurations for different resources. This feature allows you to include multiple provider blocks that refer to different configurations.

- Terraform is an immutable, declarative Infrastructure as Code language based on HashiCorp Configuration Language or JSON.

- The provider plugins downloaded by Terraform during the initialization process are stored in the `.terraform/providers` directory within the current working directory. This directory contains all the necessary provider plugins for the Terraform configuration to interact with the specified providers.



1. Add the new `resource` block to the Terraform configuration in the repo

2. Open a pull request with the change

3. After the plan is generated, approve the `apply` in HCP Terraform

This sequence follows the recommended VCS-driven workflow for deploying a new resource using HCP Terraform. By adding the new `resource` block to the Terraform configuration in the repository, opening a pull request with the change, and then approving the `apply`n b after the plan is generated, you ensure that the deployment process is controlled, reviewed, and tracked effectively.



- **Objective 4e - Write dynamic configuration using expressions and functions**

The `merge()` function takes two or more maps and combines them into a single map. When there are duplicate keys, values from maps later in the argument list take precedence. This makes it perfect for combining common tags with resource-specific tags, with the latter overriding the former if needed.


- **Objective 4d - Understand and use complex types**

In Terraform, a map(string) represents a collection of key-value pairs where both the keys and the values are strings. This data type is commonly used for storing configurations, tags, or other sets of string-based information that require access via a unique identifier.

Declaring a `map(string)` variable:

To define a variable of type map(string), you specify `type = map(string)` within the `variable` block. You can also provide a `default` value with key-value pairs.

```hcl
variable "region_amis" {  
  description = "A map of AWS regions to their corresponding AMI IDs."
  type        = map(string)
  default = {
    "us-east-1" = "ami-0c55b159cbfafe1f0",   
    "us-west-1" = "ami-0bdb828fd58c52235"  
  }
}
```

To read a value from a map, use bracket `(index)` syntax with the key as a string: `var.metadata["build-tag"]`. Dot syntax like `var.metadata.build-tag` does not work here because `build-tag` contains a hyphen, and Terraform would try to interpret it as the expression build minus tag. Bracket syntax also handles keys with spaces, dots, or other characters that are not valid identifiers.


- **Objective 4g - Validate configuration using custom conditions**

Postconditions are evaluated after a resource is created or updated and can verify that the resource was created with expected attributes. Since you need to check the instance's public IP after creation, a `postcondition` is appropriate. You would add it to the lifecycle block with a condition like `self.network_interface[0].access_config[0].nat_ip != ""`. If the condition fails, Terraform will error and the operation fails, ensuring you don't proceed with infrastructure that doesn't meet your requirements.


- **Objective 5b - Describe variable scope within modules**

Child modules have **no implicit access** to the parent’s variables, locals, or outputs. Pass inputs explicitly in the module block; outputs flow **upward** from child to parent, not the other way. Example where count and image are values that are being passed into the module and would be accessible by the child module.


```
module "web" {
  <module-specific-inputs>
  source = "./modules/web"
 
  count = var.num_vms
  image = data.aws_ami.ubuntu.id
}
```


- **Objective 7a - Import existing infrastructure into your Terraform workspace**

**NOTE: Terraform does have the ability to import using an `import` block. However, for the Terraform Associate 004 exam, Objective 7a lists the `terraform import` command explicitly, so you should be familiar with both.**

The current implementation of Terraform import can only import resources into state. It does not generate a configuration. Because of this, and before running `terraform import`, it is necessary to manually write a `resource` configuration block for the resource to which the imported object will be mapped.

First, add the resources to the configuration file:
```hcl
resource "aws_instance" "example" {
  # ...instance configuration...
}
```
Then run the following command:

`$ terraform import aws_instance.example i-abcd1234`


- The `.terraform.lock.hcl` file is a dependency lock file used by Terraform. It is created or updated every time you run `terraform init`.


**The relationship rules that usually trip people up:**

- Data flows down through input variables, and up through outputs. That's the only channel. It's the same as a function: arguments in, return value out.
- A child module cannot reference anything in the root. No `var.something_from_root`, no `module.other.x`. It only sees its own variable values and its own resources.
- Two sibling child modules can't talk to each other directly either. The root reads `module.a.id` and passes it into `module.b` as an argument. The root is the wiring.
- A resource inside a child is `module.rg.azurerm_resource_group.this` in state, but you can't address it that way in config — if you need it, output it.
- Run `terraform init` after adding or changing a module block; Terraform has to fetch/link the source before plan works.

Nesting is just this repeated — a child module can itself call another child module, 
developer.hashicorp.com and that grandchild's outputs have to be re-exported by the middle module before the root can see them.
