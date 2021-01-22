# Terraform (TF):

_A set of tools, by Hashicorp, that allow you to declaratively define your infrastructure with code and deploy that infrastructure predictably. It allows you to build, modify, and version your infrastructure safely and efficiently._

Any infrastructure, with few exceptions, managed with Terraform should be modified only through Terraform. This makes it easy to rely on the accuracy of your Terraform definitions, but it's also necessary in order to ensure your changes a persisted with future deploys.

Terraform uses UTF-8 encoded plain text files with the `.tf` file extension, also called configuration files. These files are written in Terraform's configuration language, which is based on a more general language called HCL.

## Syntax

### Blocks:

_A container for other content (resource, input variable, output value, data source, etc.)_

Contains zero or more arguments.

```
resource "aws_vpc" "main" {
  cidr_block = "10.0.0.0/16"
}
```

### Arguments:

_A mapping of key to value, applied to the block in which they appear._

```
image_id = "abc123"
```

### Expressions:

_A representation of a value, typically by reference and/or combination of other values. They appear as values for arguments or within other expressions._

```
cidr     = var.cidr
```

#### Types:

- **string**: a sequence of Unicode characters representing some text, like "hello".
- **number**: a numeric value. The number type can represent both whole numbers like 15 and fractional values like 6.283185.
- **bool**: a boolean value, either true or false. bool values can be used in conditional logic.
- **list (or tuple)**: a sequence of values, like ["us-west-1a", "us-west-1c"]. Elements in a list or tuple are identified by consecutive whole numbers, starting with zero.
- **map (or object)**: a group of values identified by named labels, like {name = "Mabel", age = 52}.

#### Strings

Strings are the most complex kind of literal expression in Terraform, with special escape sequences, heredoc syntax, interpolation, and template directives. See more here: https://www.terraform.io/docs/configuration/expressions/strings.html

### Functions:

_Built-in utilities that you can call from within expressions to transform and combine values._

The general syntax for function calls is a function name followed by comma-separated arguments in parentheses:

```
max(5, 12, 9)
```

Template

## Modules

_A collection of .tf files kept together in a directory. These can be used to avoid repetitive work._

Only top-level `.tf` files are considered part of the same module definition. These config files are evaluated together, effectively as a single document. Any nested directories within a module are treated as separate modules.

A module can reference other modules, local or external, using module calls to explicitly include them in the configuration.

External modules can be retrieved from [Terraform Registry](https://registry.terraform.io/) or other similar public registries.

Terraform always runs in the context of a single root module, with the complete configuration consisting of that root module, any dependency modules called by that root modules, and any dependencies called by those dependency modules, until the tree of modules are resolved.

While it's possible to create modules with complex branches and conditional logic, in most cases, it's strongly recommended to keep the module tree flat, with only one level of child modules, and to compose your module of expressions that describe the relationships between the child modules.

E.g.

```.tf
module "network" {
  source = "./modules/aws-network"

  base_cidr_block = "10.0.0.0/16"
}

module "consul_cluster" {
  source = "./modules/aws-consul-cluster"

  vpc_id     = module.network.vpc_id
  subnet_ids = module.network.subnet_ids
}
```

## Input Variables

_Input variables serve as parameters for a Terraform module, allowing aspects of the module to be customized without altering the module's own source code, and allowing modules to be shared between different configurations._

When you declare variables in the root module of your configuration, you can set their values using CLI options and environment variables. When you declare them in child modules, the calling module should pass values in the module block.

## Local Values

_A local value assigns a name to an expression, so you can use it multiple times within a module without repeating it._

```
locals {
  # Common tags to be assigned to all resources
  common_tags = {
    Service = local.service_name
    Owner   = local.owner
  }
}

resource "aws_instance" "example" {
  # ...

  tags = local.common_tags
}
```

## State

_A json definition of your infrastructure implementation at present, including sensitive information. This can be found at `.terraform/terraform.tfstate`._

It's recommended to store this file, encrypted, in a shared storage, to make it easy for multiple team members to work on infrastructure changes at once without overwriting each others changes. S3 is a good option for this. DynamoDB can also be used to control lock access to the file, to prevent multiple people from applying changes to the infrastructure at the same time.

E.g.

```
terraform {
    required_version = ">= 0.11.7"
    backend "s3" {
            encrypt			= true
            bucket			= "bucket-with-terraform-state"
            dynamodb_table	= "terraform-state-lock"
            region			= "us-east-1"
            key				= "locking_states/terraform.tfstate"
    }
}
```

If you have multiple environments, it's recommended to split up your Terraform state by environment and even component, to allow multiple collaborators to work on different parts of the infrastructure at the same time.

This can be achieved using the specific key component in the Backend definition:

```
# my_infra/prod/database/main.tf:
		...
		key			= "prod/database/terraform.tfstate"
		...
# my_infra/dev/database/main.tf:
		...
		key			= "dev/database/terraform.tfstate"
		...
# my_infra/dev/loadbalancer/main.tf:
		...
		key			= "dev/loadbalancer/terraform.tfstate"
		...
```

## Terraform CLI:

_Terraform provides a CLI, which allows you to manage your local and external terraform setups as well as provision infrastructure into a cloud environment. The instructions can be found here: https://learn.hashicorp.com/tutorials/terraform/install-cli._

A common flow for provisioning your infrastructure is:

```bash
terraform get
terraform plan -input=false -var-file=ecs.tfvars
terraform apply -input=false -var-file=ecs.tfvars
```

In this example, we assume we've stored variables for our configuration in a file called `ecs.tfvars`.

### `terraform get`

_The `terraform get` command is used to download and update modules mentioned in the root module (https://www.terraform.io/docs/commands/get.html)._

**Note:** When installing a remote module, Terraform will download it into the .terraform directory in your configuration's root directory. When installing a local module, Terraform will instead refer directly to the source directory. Because of this, Terraform will automatically notice changes to local modules without having to re-run terraform init or terraform get.

### `terraform plan -input=false -var-file=ecs.tfvars`

_The `terraform plan` command is used to create an execution plan. This command is a convenient way to check whether the execution plan for a set of changes matches your expectations without making any changes to real resources or to the state. For example, `terraform plan` might be run before committing a change to version control, to create confidence that it will behave as expected (https://www.terraform.io/docs/commands/plan.html)._

`-input=false` specifies that we don't want prompted for input for variables not directly set.

`-var-file=ecs.tfvars` specifies that we want to specify variables in our terraform configuration from the **ecs.tfvars** file.

### `terraform apply -input=false -var-file=ecs.tfvars`

_The `terraform apply` command is used to apply the changes required to reach the desired state of the configuration. (https://www.terraform.io/docs/commands/apply.html)._

`-input=false` specifies that we don't want prompted for input for variables not directly set.

`-var-file=ecs.tfvars` specifies that we want to specify variables in our terraform configuration from the **ecs.tfvars** file.

## Outputs

_A block used to show the information needed after Terraform templates are deployed. They are also used within modules to export information._

When used within modules, two outputs must be defined, one in the module and a similar one in the configuration files. Output information is stored in a Terraform state file and can be queried by other terraform templates.

It's common to organize module outputs in a purpose-built file, `outputs.tf`.

## Datasource

_A special kind of resource block used to request data from an external source and expose it locally under a given name._

For example, below we are getting the latest Linux 2 ECS-optimized AMI, which can later be referenced by other resources as `data.aws_ami.latest_ecs_ami`:

```
data "aws_ami" "latest_ecs_ami" {
  most_recent = true

  filter {
    name   = "name"
    values = ["amzn2-ami-ecs-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }

  owners = ["amazon"]
}
```

## Resources

_Resources are the most important element in the Terraform language. Each resource block describes one or more infrastructure objects._

For example, below we are declaring an aws_instance resource and naming it "web", which can later be referenced by other resources in this module as `aws_instance.web`.

```
resource "aws_instance" "web" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}
```

Each resource has a single resource type, and each resource type is implemented by a provider, which is a plugin for Terraform that offers a collection of resource types.

## Provider

_A plugin for Terraform that offers a collection of resource types and/or data sources that Terraform can manage._

In order to manage resources, a Terraform module must specify which providers it requires. Additionally, most providers need some configuration in order to access their remote APIs, and the root module must provide that configuration.

For example, below we are configuring an aws provider, authenticating to the aws profile "example", which we've defined via the AWS CLI:

```
provider "aws" {
  region  = "us-east-1"
  profile = "example"
}
```

Terraform usually automatically determines which provider to use based on a resource type's name. (By convention, resource type names start with their provider's preferred local name.)

```
resource "aws_instance" "web" {
  ami           = "ami-a1b2c3d4"
  instance_type = "t2.micro"
}
```

When using multiple configurations of a provider (or non-preferred local provider names), you must use the provider meta-argument to manually choose an alternate provider configuration. See below.

```
# default configuration
provider "google" {
  region = "us-central1"
}

# alternate configuration, whose alias is "europe"
provider "google" {
  alias  = "europe"
  region = "europe-west1"
}

resource "google_compute_instance" "example" {
  # This "provider" meta-argument selects the google provider
  # configuration whose alias is "europe", rather than the
  # default configuration.
  provider = google.europe

  # ...
}
```
