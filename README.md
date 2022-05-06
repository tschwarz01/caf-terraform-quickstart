# caf-terraform-quickstart


# Components


## [Aztfmod](https://github.com/aztfmod/terraform-azurerm-caf)
- The Cloud Adoption Framework Terraform Supermodule
- Can be used in a standalone fashion or with Rover

## [Rover](https://github.com/aztfmod/rover)

**Docker container**
- Allows consistent developer experience on PC, Mac, Linux, including the right tools, git hooks and DevOps tools.
- Native integration with Visual Studio Code, GitHub Codespaces.
- Contains the versioned toolset you need to apply landing zones.
- Helps you to switch components versions fast by separating the run environment from the configuration environment.

**Terraform wrapper**
- Manages Terraform state files on Azure storage account for multiple Terraform deployments.
- Facilitates the transition to CI/CD.
- Enables seamless experience (state connection, execution traces, etc.) locally and inside pipelines.
- Is available on Docker Hub as a standalone image, or as a Pipeline agent which supports Azure DevOps, GitHub Actions, Gitlab & Terraform Cloud/Terraform Enterprise


## [CAF Platform Starter Kit](https://github.com/Azure/caf-terraform-landingzones-platform-starter)
- The platform starter project is an empty development environment which comes preloaded with all the utilities needed for deploying landing zones at scale

## [CAF Landing Zones](https://github.com/Azure/caf-terraform-landingzones)
- Contains full solution accelerators, individual templates, Ansible playbooks and example CICD pipeline configuration tasks


---

# Quickstart

### **Deploying the Rover Launchpad & example CAF "Platform Landing Zones"**

Follow the instructions in the Getting Started guide - https://aztfmod.github.io/documentation/docs/azure-landing-zones/landingzones/alz-intro

### **Using Rover for a custom deployment**

Use this guide if
- your experience with Terraform mostly consists of developing locally on your own computer using an IDE such as Visual Studio Code.
- you don't have full access to create Azure subscriptions, or you don't have full access to Azure AD
- you are intrigued by the idea of leveraging state files across multiple Terraform deployments
- you are feeling overwhelmed by the full platform starter guide

The following guide will help you
- setup your development environment.
- become familiar with the directory structure.
- understand common components, such as the launchpad, template, definition and configuration files.

## **Local Environment Setup**

The first tasks you will need to perform involve installing required utilities to your development machine.
- Azure CLI: https://docs.microsoft.com/en-us/cli/azure/install-azure-cli
- IDE - We'll be using VSCode: https://code.visualstudio.com/download
- If using a Windows OS, you will need to configure Windows Subsystem for Linux and install Docker: https://docs.microsoft.com/en-us/windows/wsl/tutorials/wsl-containers
- Remote Development Extension Pack for VSCode: https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.vscode-remote-extensionpack

Create forks of the following GitHub repositories
- https://github.com/Azure/caf-terraform-landingzones-platform-starter
- https://github.com/Azure/caf-terraform-landingzones

We are now ready to begin.  Follow the next steps in the prescribed order.
1. Clone your fork of the platform starter repo to your local machine
2. Change directory into the freshly cloned local repository
3. Open VSCode
```bash
git clone https://github.com/[YOUR_ACCOUNT]/caf-terraform-landingzones-platform-starter
cd caf-terraform-landingzones-platform-starter
code .
```
4. VSCode should prompt you with a message stating 'Folder contains a Dev Container configuration file...', select Reopen in Container at this prompt.  The container may take a short time to fully initialize.  When it is complete, you should have a bash or zsh prompt visible in the terminal.
5. Take a moment to look around.  You should notice a few things.

There's not much here at the moment.  You should see only two directories containing configuration settings for the devcontainer and vscode.  Additionally, you'll see common markdown files & git related configuration files.

In the terminal, if you type "`pwd`", you'll see that the current directory context is `/tf/caf`

If you type "`ls /`" you'll see the directories and directory structure you'd expect from an Ubuntu based VM.

Feel free to explore. You'll notice that there several utilities which come pre-installed in this image, including Terraform, Ansible, Git and others. When you finish exploring, continue with step 6 below.

6. From the `/tf/caf` directory, clone the caf-terraform-landingzones fork you created previously to the /tf/caf/terraform directory.



```bash
git clone https://github.com/[YOUR_ACCOUNT]/caf-terraform-landingzones landingzones
```
7. Change your working directory via "`cd /tf/caf/landingzones`"
8. We may need to checkout a different working branch from the caf-terraform-landingzones.

> Example: `git checkout 2204.1.int`

> NOTE: The platform-starter-kit walkthrough often instructs users to checkout the latest YYMM branch.
At present, I've been using the branch 2204.1.int.  If this branch is no longer available when you are reading this, you can use "`git branch -r`" to view the list of remote branches.

That's it!  We are ready to begin composing our deployments. However, we have a decision to make.  We can either
-- use the included CAF templates to deploy the example Platform Landing Zones.
or
-- customize our deployment to our specific needs.

---

## **Option A: Deploy the Platform Landing Zones using the provided templates** ##

---

If you've read through the platform-starter walkthough, you'll have noticed that we have not yet departed from the instructions it provides.  In fact, if you want to deploy all of the resources used by the Platform Landing Zone subscriptions, you could do that at this time by executing the deployment bash script located at `/tf/caf/landingzones/templates/platform/deploy_platform.sh`.  However, there are multiple drawbacks to starting with this approach, including:
- Requires privileged access to both Azure and Azure AD
- It is possible to deploy into a MSDN subscription; however, when fully deployed, it incurs significant costs
- It takes a significant amount of time to walk through and execute the many nested deployments required for this scenario
- The deployment is largely pre-configured, and offers few opportunities to increase your understanding of how rover works or how templates, definition files and configuration files are related.

>  In my opinion, it is this last bullet point that is most significant.  It required a huge time investment to reverse engineer how the scripts and templates interact to produce the final product.

If you want to continue with this approach, refer to the walkthrough which begins here: https://aztfmod.github.io/documentation/docs/azure-landing-zones/landingzones/platform/single%20reuse/elsz-single-reuse

---

## **Customized Deployment** ##

---

A second possible course to take from this point would be to deploy the launchpad ("`/tf/caf/landingzones/caf_launchpad`") & one or more of the scenarios defined at "`/tf/caf/landingzones/caf_solution/scenario`".  This is a good starter option, and we will build upon it via reusing the  **launchpad** and the **foundations** deployments for the final two example deployments in our customized solution.

## **Initializing Rover** ##
The first command you will have to run is ```rover login```:

You can run a plain rover login:

```bash
rover login
```

You can specify additional context to restrict the token, like the tenant name and subscription to use:

```bash
rover login --tenant [tenant_name.onmicrosoft.com or tenant_guid (optional)] --subscription [subscription_id_to_target(optional)]
```

You can log out running the following command:

```bash
rover logout
```


## **Launchpad** ##

First, what is **launchpad**?
- The **launchpad** is a base level construct, deployed prior to any other resources, which consists of the storage accounts, Key Vault, RBAC assignments, general configuration settings such as which Azure regions will be targeted by your deployment, and components related to Terraform state management.
- The **launchpad** resources are utilized primarily by the Rover utility.  Rover will automatically store & retrieve deployment state files as needed.

As a last step prior to deploying the **launchpad**, I recommend that you look at the contents of its "`configuration.tfvars`" file, which can be found at "`/tf/caf/landingzones/caf_launchpad/scenario/100/configuration.tfvars`".  Take notice that the "`landingzone`" map object contains only three properties, none of which is named tfstate.  The significance of this will become more apparent as we move forward.

**landingzone** map object from **launchpad configuration.tfvars**
```json
landingzone = {
  backend_type = "azurerm"
  level        = "level0"
  key          = "launchpad"
}
```

>  Personal preference: If I plan to make changes to the **launchpad** configuration files, I first create a new directory via "`mkdir /tf/caf/configuration`", then copy the "`caf_launchpad`" directory to "`/tf/caf/configuration`".  The same is true if modifying the **foundations** configuration files, which we'll discuss shortly.

## **Deploying Launchpad** ##

Next, we need to deploy the launchpad:

```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_launchpad -a apply \
            -var-folder /tf/caf/landingzones/caf_launchpad/scenario/100 \
            -level level0 \
            -launchpad \
            -parallelism=30 \
            --environment contoso \
            '-var prefix=cont'
```

We can look in the Azure portal to confirm that several resource groups were deployed, with each containing a storage account and keyvault.

## **Deploying foundations** ##

The **foundations** scenario can be found at "`/tf/caf/landingzones/caf_solution/scenario/foundations/100-passthrough`".  This directory contains a single file, which itself contains a single map object that should look somewhat familiar.

>  NOTE: No resources are created by this deployment.

**foundations** scenario **landingzone.tfvars**
```json
landingzone = {
  backend_type        = "azurerm"
  global_settings_key = "launchpad"
  level               = "level1"
  key                 = "caf_foundations"
  tfstates = {
    launchpad = {
      level   = "lower"
      tfstate = "caf_launchpad.tfstate"
    }
  }
}
```

You'll recall we also had a **landingzone** map object in the **launchpad** configuration, but this one has an additional map property: **tfstates**.
>You will almost always see any deployment that is not the **launchpad** itself will have this property.

It's purpose is to define what state dependencies exist between the current configuration, and resources which were defined by separate Terraform deployments with remote state files.  In this case, we are saying that the **foundations** deployment has a dependency on the **launchpad** deployment, which is the name of the key used within tfstate map object.

The **launchpad** map key contains two additional properties: **level** and **tfstate**.  With level, we are stating that the launchpad deployment occurred at a lower level in the hierarchy than the current configuration is targeting - which we can see via the landingzone.**level** property in the object map.  **Launchpad** resides at **level0** and **caf_foundations** resides at **level1**.

To deploy this configuration, execute the following:
```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_solution -a apply \
            -var-folder /tf/caf/landingzones/caf_solution/scenario/foundations/100-passthrough \
            -tfstate caf_foundations.tfstate \
            -level level1 \
            -parallelism=30 \
            --environment contoso
```

---

## **Custom Deployment** ##

---

We are ready to begin composing the resource definitions which describe the resources we want to deploy to Azure.  We will break this up into two deployments to demonstrate linked dependencies, which I have creatively named **test** and **test2**.

Before we start, if you did not create a configuration folder previously, create one now at "`/tf/caf/configuration`".

> **Levels Hierarchy:**
If you refer to the documentation related to **levels hierarchy** at https://aztfmod.github.io/documentation/docs/fundamentals/lz-intro, you will see that level0 through level2 is generally reserved for the Platform Landing Zone configuration.  With that in mind, we need to create a **level3** folder at "`/tf/caf/configuration/level3`", then subdirectories for `test` and `test2`.

### **"test" deployment** ###
Beginning with **test**, create a file named **configuration.tfvars** at "`/tf/caf/configuration/level3/test/configuration.tfvars`".  Insert the following content:

```json
global_settings = {
  default_region = "region1"
  regions = {
    region1 = "westus3"
    region2 = "centralus"
    region3 = "southcentralus"
  }
}
landingzone = {
  backend_type        = "azurerm"
  level               = "level1"
  key                 = "testing_level1"
  global_settings_key = "launchpad"
  tfstates = {
    launchpad = {
      tfstate   = "caf_launchpad.tfstate"
      level     = "lower"
    }
  }
}
resource_groups = {
  # Default to var.global_settings.default_region. You can overwrite it by setting the attribute region = "region2"
  evh_examples = {
    name   = "eventhub"
    region = "region1"
  }
}
event_hub_namespaces = {
  evh1 = {
    name               = "evh1"
    resource_group_key = "evh_examples"
    sku                = "Standard"
    region             = "region1"
  }
}
```
Again, we're working with map objects that define configuration settings or Azure resources.

Let's step through them one at a time:

**global_settings**
```json
global_settings = {
  default_region = "region1"
  regions = {
    region1 = "westus3"
    region2 = "centralus"
    region3 = "southcentralus"
  }
}
```

This is relatively straight forward.  We are declaring a default region for our resource deployment via its key - **region1**.  The key maps to the region definition in the **regions** map.  In this case, **region1** maps to the region **westus3**.

**landingzone**
```json
landingzone = {
  backend_type        = "azurerm"
  level               = "level1"
  key                 = "testing_level1"
  global_settings_key = "launchpad"
  tfstates = {
    launchpad = {
      tfstate   = "caf_launchpad.tfstate"
      level     = "lower"
    }
  }
}
```
We've explored this object previously.
- We are definining this as a **level1** deployment.
- The landing zone key, which we'll see used in our **test2** example, is set to **testing_level1**.

We have a **tfstates** map
- **tfstates** defines that we are taking a dependency on **launchpad**,
- **launchpad** uses a tfstate file named **caf_launchpad.tfstate**
- **caf_launchpad.tfstate** is located in the level below / **lower** than the current configuration.

**resource_groups**
```json
resource_groups = {
  # Default to var.global_settings.default_region. You can overwrite it by setting the attribute region = "region2"
  evh_examples = {
    name   = "eventhub"
    region = "region1"
  }
}
```
Finally, the first resource we're defining ourselves that will be deployed to Azure!  Let's deconstruct it:
- The map key is **evh_examples**
- The name we are assigning to this resource group is **eventhub**
- The resource group should be placed in **region1**

What is region1?  The aztfmod module knows that we defined our list of potential regions in an input parameter named global_settings.regions.  The module stores that in a variable when this configuration is executed.  Therefore, it can access the value of the region key by referencing var.global_settings.regions["**region1**"].value, which in this case would return the value **westus3**.

Lastly, we have a final map:
**event_hub_namespaces**
```json
event_hub_namespaces = {
  evh1 = {
    name               = "evh1"
    resource_group_key = "evh_examples"
    sku                = "Standard"
    region             = "region1"
  }
}
```

- key is **evh1**
- ...

The same principles apply here to **resource_group_key** and **region**.  The keys specified will be used to lookup our input values in the **resource_groups** and **global_settings.regions** maps, respectively.

### **Execution** ###
To deploy these resources, we execute the following **Rover** command:
```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_solution/ -a apply \
            -tfstate testing_level1.tfstate \
            -level level1 \
            -parallelism=30 \
            -var-folder /tf/caf/configuration/level1/test \
            --environment contoso
```
> NOTE: We are specifying the name of our tfstate, level and referencing the folder containing our tvfars configuration file for this deployment

That's it for our first deployment: **test**

### **"test2" deployment** ###

Create a file named **configuration.tfvars** at "`/tf/caf/configuration/level3/test2/configuration.tfvars`".  Copy the following code into the file:

**configuration.tfvars**
```json
landingzone = {
  backend_type        = "azurerm"
  level               = "level1"
  key                 = "testing_level1_storage"
  global_settings_key = "launchpad"
  tfstates = {
    launchpad = {
      tfstate   = "caf_launchpad.tfstate"
      level     = "lower"
    }
    testing_level1 = {
      tfstate = "testing_level1.tfstate"
    }
  }
}
storage_accounts = {
  nsgflogs = {
    name                      = "nsglogs"
    resource_group = {
    key        = "evh_examples"
    lz_key = "testing_level1"

    }
    account_kind              = "BlobStorage"
    account_tier              = "Standard"
    shared_access_key_enabled = false
    account_replication_type  = "LRS"
  }
}
```

Let's deconstruct:

**landingzone**
```json
landingzone = {
  backend_type        = "azurerm"
  level               = "level1"
  key                 = "testing_level1_storage"
  global_settings_key = "launchpad"
  tfstates = {
    launchpad = {
      tfstate   = "caf_launchpad.tfstate"
      level     = "lower"
    }
    testing_level1 = {
      tfstate = "testing_level1.tfstate"
    }
  }
}
```
The only changes from our **test** deployment are:
- **key** now defines this as a separate deployment named **testing_level1_storage**
- We've defined a new map under tfstates indicating that, in addition to the **launchpad** state, we have dependencies on a state with key of **testing_level1**.  If you recall, this corresponds with the **landingzone.key** value we assigned to our **test** deployment.

**storage_accounts**
```json
storage_accounts = {
  nsgflogs = {
    name                      = "nsglogs"
    resource_group = {
    key        = "evh_examples"
    lz_key = "testing_level1"

    }
    account_kind              = "BlobStorage"
    account_tier              = "Standard"
    shared_access_key_enabled = false
    account_replication_type  = "LRS"
  }
}
```
Here we have a new map type named **storage_accounts**, which is exactly what you think it is.  Within storage_accounts, we're defining the following:
- map key is **nsgflogs**
- the name of our storage account will be **nsglogs**
- ignore the resource_group map for now
- define a few more common storage account attributes

The interesting piece here is the **resource_group** map within the **storage_accounts.nsgflogs** object.
- **resource_group.key** - This is the key for a map object in our **resource_groups** map; however, we did not declare any resource groups in this configuration.
- **lz_key** - We are referencing **landingzone.key** of our **test** deployment.

Putting that information together, we are declaring that we want this storage account deployed into a resource group managed by a remote Terraform deployment (**testing_level1**).  Because we declared a tfstate dependency on **testing_level1**, rover & the aztfmod module will work together to bring in all of the resource information contained within **testing_level1's** tfstate.  The resource information will be stored in local variables within our current deployment context.  The module can then reference the properties it needs from the **test** deployment resource group identified by the key **evh_examples** like this: `local.combined_objects_resource_groups["testing_level1"]["evh_examples"].PROPERTY_NAME`

This configuration can be deployed by executing the following rover command:
```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_solution/ -a apply \
            -tfstate testing_level1_storage.tfstate \
            -level level1 \
            -parallelism=30 \
            -var-folder /tf/caf/configuration/level1/test2 \
            --environment contoso
```

When complete, you will find the storage account you defined in the **test2** deployment located within the resource group created by deployment **test**.

---
## Wrap up ##
---

We've seen how to leverage the CAF Terraform Supermodule with the Rover toolset to deploy modularized Terraform configurations with interdependencies.
