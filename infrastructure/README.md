# **OCTO : `azure-tf-library-mono`**

# Modules

This repository includes a library of curated [Terraform](https://registry.terraform.io/providers/hashicorp/azurerm/latest) modules as well as a Continuous Integration (CI) environment using [GitHub Actions](https://github.com/features/actions) for modules' validation.

# Requirements

Use cases the proposed structure will cover

- Azure as a cloud provider. This could also be applied to others as Terraform supports many different providers.
- Azure multi-subscription. One account for production workloads and one for testing purposes.
- Crucial resources (e.g. database) are deployed manually and on demand.
- Short-lived dynamic environments to support review apps for branches.
- Sharing and referencing resources created by other Terraform configurations.
- Frictionless local development.
- Ability to deploy from a local machine without a complex authentication flow.
- Terraform states are stored in a separate Azure core management account.

# Context

Resources are assigned to a given environment:

- **prod** (production)
- **pre** (pre-production/staging). An environment for testing purposes. Acts as a gate before the production. Mirrors in the production environment.
- **rev** (review). Dynamic short-lived environments. Spin up on demand. We can have many of them. Mimics the pre-production environment.

# Terraform Directory Structure

```bash
infrastructure/
├── environments/
│   ├── prod/
│   │   ├── config.tf
│   │   └── vpc/
│   │       ├── main.tf
│   │       ├── output.tf
│   │       ├── variables.tf
│   │       └── config.tf
│   ├── pre/
│   │   ├── config.tf
│   │   └── vpc/
│   │       ├── main.tf
│   │       ├── output.tf
│   │       ├── variables.tf
│   │       └── config.tf
│   └── rev/
│       ├── config.tf
│       └── vpc/
│           ├── main.tf
│           ├── output.tf
│           ├── variables.tf
│           └── config.tf
└── modules/
    ├── vpc/
    │   ├── main.tf
    │   ├── outputs.tf
    │   └── variables.tf
    ├── eks/
    │   ├── main.tf
    │   ├── outputs.tf
    │   └── variables.tf
    └── rds/
        ├── main.tf
        ├── outputs.tf
        └── variables.tf
```

*Modules are reusable Terraform configurations that can be called and configured by other configurations.*

Local Machine Deployment

```bash
$ cd infrastructure/environments/pre
$ terraform init
$ terraform apply
$ cd ../../../infrastructure/environments/pro
$ terraform init
$ terraform apply
```

## Exploration

Going deeper into the `infrastructure` folder, you will find `environments` and `modules`*.* Inside `each environment`, we have a separate directory for each environment. In `modules`, you will find Terraform modules imported by at least two environments (DRY 😉). DRY = Dont Repeat Yourself

The essence** of this approach and the thing that defines why this approach is so great. You could be tempted to use a configuration-driven approach where you have one Terraform configuration, but depending on the input, it generates different results. If you have three similar environments (pre, pro, rev), why not create one Terraform module and control it with a configuration stored in a *json* file? This idea can be influenced by the config chapter from the Twelve-Factor App methodology, but it relates to an app code (environment-agnostic), not an infrastructure code (it basically describes environments). This is what the deployment of **pre** environment could look like.

```
$ terraform apply -var-file="pre.tfvars.json"
```

If you would use a configuration-driven approach, you will sooner or later end up with many conditional statements and other bizarre hacks. Terraform is a tool intended to create declarative descriptions of your infrastructure, so please do not introduce imperativeness. By preparing separate Terraform modules for each environment, we see things how they look in reality. We have a direct, explicit mapping between code and a Terraform state. We limit WTFs/minute. The proposed solution will have more lines of code but still leave space for code reuse. Don’t forget about the modules!

