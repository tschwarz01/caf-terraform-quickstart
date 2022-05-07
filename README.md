# Foundations

## The Azure CAF Terraform Solution Components
---

### [Aztfmod Repository](https://github.com/aztfmod/terraform-azurerm-caf)
- [The Cloud Adoption Framework Terraform Supermodule](https://registry.terraform.io/modules/aztfmod/caf/azurerm/latest)
- Can be used in a standalone fashion or with Rover

### [Rover Repository](https://github.com/aztfmod/rover)

**Docker container**
- Allows consistent developer experience on PC, Mac, Linux, including the right tools, git hooks and DevOps tools.
- Native integration with Visual Studio Code, GitHub Codespaces.
- Contains the versioned toolset you need to apply landing zones.
- Helps you to switch components versions fast by separating the run environment from the configuration environment.

**Terraform wrapper**
- Manages Terraform state files on Azure storage account for multiple Terraform deployments.
- Facilitates the transition to CI/CD.
- Enables seamless experience (state connection, execution traces, etc.) locally and inside pipelines.
- Is available on [Docker Hub as a standalone image](https://hub.docker.com/r/aztfmod/rover/tags?page=1&ordering=last_updated), or as a [Pipeline agent](https://hub.docker.com/r/aztfmod/rover-agent) which supports Azure DevOps, GitHub Actions, Gitlab & Terraform Cloud/Terraform Enterprise.


### [CAF Platform Starter Kit Repository](https://github.com/Azure/caf-terraform-landingzones-platform-starter)
- The platform starter project is an empty development environment which comes preloaded with all the utilities needed for deploying landing zones at scale.

### [CAF Landing Zones Repository](https://github.com/Azure/caf-terraform-landingzones)
- Contains full solution accelerators, individual templates, Ansible playbooks and example CICD pipeline configuration tasks.

## Component Interactions
---

The Terraform modules, git repositories and CLI utilities outlined above are primarily used in one of two ways.

### **standalone** mode

This approach, which directly utilizes only the Terraform CAF Supermodule, will be most familiar to Terraform developers who have experience working with the Terraform azurerm module.

Use the **standalone** approach:
- As a learning resource on how composition differs between the Terraform CAF Supermodule and the Terraform azurerm module.
- Performing local development of relatively simple Azure deployments targeting a single Azure subscription.

### **CAF Landing Zones** mode

With this approach, we leverage the **Platform Starter Kit** to provide us with the Docker development environment and version controlled tooling.  Generally, we combine the **Platform Starter Kit** with the **Landing Zones Repository** via cloning the latter repository into our Docker development environment.  Lastly, we use a variety of tools, including the **rover cli**, to customize and deploy the pre-built CAF Platform Landing Zones, as well as any additional custom landing zones we define.

Use the **CAF Landing Zones** approach:
- To create, manage and deploy complex multi-deployment configurations.
- When you have dependencies between discrete Terraform deployments and therefore need access to resource details contained within remote Terraform tfstates.

# Note from author
---
The reason this guide exists is solely due to the difficulties I personally had trying to absorb all the concepts necessary when using the walkthrough which (attempts to) show you how to deploy the most complex pre-built scenario available in this repository.  There are many Azure & Azure AD high privileged access requirements, over a dozen deployment steps with multiple parameters, each of which must executed in a certain order (order is not always clear in the walkthrough), with minimal context provided regarding what each step is doing or how it's doing it.  Perhaps most significantly, the entire walkthrough can require many hours worth of effort and, when mostly deployed, the daily costs are not insignificant if your intention is only to learn this framework for future enterprise use.

The power of this framework, in my opinion, is in its ability to manage and share state across multiple discrete Terraform deployments, along with the ability to reference both local and remote resource via dictionary key reference.  To that end, I found myself repeatedly asking the same question:  How do I use this framework with as few deployments possible to be able to leverage and demonstrate the state sharing capabilities?

The local development + custom deployment portion of this walkthrough will teach you how to accomplish that goal, and tries to provide important insight into how the framework operates so that you know how to troubleshoot issues when they occur.


## Next Steps
---

Determine if you will be developing Terraform configurations on a local development machine, or if you intend to configure CI/CD pipelines via Azure DevOps or GitHub Actions
- [**Local Environment Setup**](./local_dev/L_common_prerequisites.md)
- [**Remote Environment Setup**](./remote_dev/R_common_prerequisites.md) - work in progress
