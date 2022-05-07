# Custom CAF Deployment
## Local development

---

Start with this scenario if
- your experience with Terraform mostly consists of developing locally on your own computer using an IDE such as Visual Studio Code.
- you don't have full access to create Azure subscriptions, or you don't have full access to Azure AD
- you are intrigued by the idea of leveraging state files across multiple Terraform deployments
- you are feeling overwhelmed by the full platform starter guide

The following guide will help you
- become familiar with the directory structure.
- understand common components, such as the launchpad, template, definition and configuration files.

If you haven't completed the [**Local Environment Setup**](./local_dev/L_common_prerequisites.md) steps, do so now and then return here.


## Examining the /tf/caf/landingzones directory
---

The `/tf/caf/landingzones` directory contains the caf-terraform-landingzones repository that we cloned into our environment during the [setup steps](./local_dev/L_common_prerequisites.md).

The purpose of this repository is to provide you with a set of pre-built templates which can help accelerate deployment of the Cloud Adoption Framework Platform Landing Zones.  It contains several important subdirectories that we will briefly explore now.

| Directory	      | Components                                                                                        |
|-----------------|---------------------------------------------------------------------------------------------------|
| caf_launchpad   | Contains the base Terraform template for deploying the **launchpad** resources that are used by the **rover** utility for secrets & state management.  There are two bootstrap scenarios available for use with the base template, the code for which is contained within the `caf_launchpad/scenarios` subdirectories.
| caf_solution    | Contains the base Terraform template and many more advanced solution accelerators pre-built for common scenarios.  These scenarios are located at `caf_solution/scenarios`.
| documentation   | Contains general instructions for configuring CI/CD pipelines in Azure DevOps or GitHub Actions.
| templates       | Contains several important subdirectories we'll explore next.  The content beneath this directory is used for deploying more complex scenarios, such as the CAF Platform Landing Zones.

| templates	directory | Components                                                                                        |
|-----------------|---------------------------------------------------------------------------------------------------|
| ansible         | Contains the Ansible playbook configuration files which are used to generate solution definition files for more complex scenarios, such as the CAF Platform Landing Zones and custom landing zones.
| asvm            | Short for Azure Subscription Virtual Machine.  This directory contains one solution example, named Orion, containing the Ansible playbook files needed for generating a custom landing zone.
| platform        | Contains the full YAML based definition files for the CAF Platform Landing Zone scenario.  Also contains Jinja resource templates needed by the YAML definition files.
| resources       | Contains the Jinja definition files for a variety of Azure resource types.  These files are used to transform the environment YAML definitions into Terraform tfvar input files.

We will be using a small subset of content within the `/tf/caf/landingzones` directory tree for our first custom deployment.

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

### What is the **launchpad**?
- The **launchpad** is a base level construct, deployed prior to any other resources, which consists of the storage accounts, Key Vault, RBAC assignments, general configuration settings (such as which Azure regions will be targeted by your deployment), and components related to Terraform state management.
- The **launchpad** resources are utilized primarily by the Rover utility.  Rover will automatically store & retrieve deployment state files as needed.

As a last step prior to deploying the **launchpad**, I recommend that you look at the contents of its `configuration.tfvars` file, which can be found at `/tf/caf/landingzones/caf_launchpad/scenario/100/configuration.tfvars`.  Take notice that the **landingzone** map object contains only three properties, none of which is named tfstate.  The significance of this will become more apparent as we move forward.

**landingzone** map object from **launchpad configuration.tfvars**
```js
landingzone = {
  backend_type = "azurerm"
  level        = "level0"
  key          = "launchpad"
}
```

>  Personal preference: If I plan to make changes to the **launchpad** configuration files, I first create a new directory via "`mkdir /tf/caf/configuration`", then copy the "`caf_launchpad`" directory to "`/tf/caf/configuration`".  The same is true if modifying the **foundations** configuration files, which we'll discuss shortly.

### **Deploying the Launchpad** ###

Next, we need to deploy the launchpad.  This can be achieved by executing the following command within the VSCode terminal.

```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_launchpad -a apply \
            -var-folder /tf/caf/landingzones/caf_launchpad/scenario/100 \
            -level level0 \
            -launchpad \
            -parallelism=30 \
            --environment contoso \
            '-var prefix=cont'
```

Here we are specifying in the previous command that the **level** for this command is **level0**.  If you're unfamiliar with the hierarchy of levels, don't worry.  We will cover the topic in one of the upcoming sections of this document.

We can look in the Azure portal to confirm that several resource groups were deployed, with each containing a storage account and keyvault.

Next we'll deploy the foundations scenario.

## **Deploying foundations** ##

The **foundations** scenario can be found at "`/tf/caf/landingzones/caf_solution/scenario/foundations/100-passthrough`".  This directory contains a single file, which itself contains a single map object that should look somewhat familiar.

>  NOTE: No resources are created by this deployment.

**foundations** scenario **landingzone.tfvars**
```js
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

Now that we've seen how to configure a Terraform deployment to take a dependency on a remote deployment's tfstate, we'll move on to a more concrete example demonstrating how this can be useful.


## **Custom Deployment** ##

---

We are ready to begin composing the resource definitions which describe the resources we want to deploy to Azure.  We will break this up into two deployments to demonstrate linked dependencies, which I have creatively named **testconfig1** and **testconfig2**.

Before we start, if you did not create a configuration folder previously, create one now at "`/tf/caf/configuration`".

> **Levels Hierarchy:**
If you refer to the documentation related to **levels hierarchy** at https://aztfmod.github.io/documentation/docs/fundamentals/lz-intro, you will see that level0 through level2 is generally reserved for the Platform Landing Zone configuration.  With that in mind, we need to create a **level3** folder at "`/tf/caf/configuration/level3`", then subdirectories for `testconfig1` and `testconfig2`.

### **"test" deployment** ###
Beginning with **testconfig1**, create a file named **configuration.tfvars** at "`/tf/caf/configuration/level3/testconfig1/configuration.tfvars`".  Insert the following content:

```js
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
  key                 = "test_config1"
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
```js
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
```js
landingzone = {
  backend_type        = "azurerm"
  level               = "level1"
  key                 = "test_config1"
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
- The landing zone key, which we'll see used in our **testconfig2** example, is set to **test_config1**.

We have a **tfstates** map
- **tfstates** defines that we are taking a dependency on **launchpad**,
- **launchpad** uses a tfstate file named **caf_launchpad.tfstate**
- **caf_launchpad.tfstate** is located in the level below / **lower** than the current configuration.

**resource_groups**
```js
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
```js
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
            -tfstate test_config1.tfstate \
            -level level1 \
            -parallelism=30 \
            -var-folder /tf/caf/configuration/level1/testconfig1 \
            --environment contoso
```
> NOTE: We are specifying the name of our tfstate, level and referencing the folder containing our tvfars configuration file for this deployment

That's it for our first deployment: **testconfig1**

### **"testconfig2" deployment** ###

Create a file named **configuration.tfvars** at "`/tf/caf/configuration/level3/testconfig2/configuration.tfvars`".  Copy the following code into the file:

**configuration.tfvars**
```js
landingzone = {
  backend_type        = "azurerm"
  level               = "level1"
  key                 = "test_config2_storage"
  global_settings_key = "launchpad"
  tfstates = {
    launchpad = {
      tfstate   = "caf_launchpad.tfstate"
      level     = "lower"
    }
    testing_level1 = {
      tfstate = "test_config1.tfstate"
    }
  }
}
storage_accounts = {
  nsgflogs = {
    name                      = "nsglogs"
    resource_group = {
    key        = "evh_examples"
    lz_key = "test_config1"

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
```js
landingzone = {
  backend_type        = "azurerm"
  level               = "level1"
  key                 = "test_config2_storage"
  global_settings_key = "launchpad"
  tfstates = {
    launchpad = {
      tfstate   = "caf_launchpad.tfstate"
      level     = "lower"
    }
    testing_level1 = {
      tfstate = "test_config1.tfstate"
    }
  }
}
```
The only changes from our **testconfig1** deployment are:
- **key** now defines this as a separate deployment named **test_config2_storage**
- We've defined a new map under tfstates indicating that, in addition to the **launchpad** state, we have dependencies on a state with key of **test_config1**.  If you recall, this corresponds with the **landingzone.key** value we assigned to our **testconfig1** deployment.

**storage_accounts**
```js
storage_accounts = {
  nsgflogs = {
    name                      = "nsglogs"
    resource_group = {
    key        = "evh_examples"
    lz_key = "test_config1"

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
- As we've specified a **lz_key**, we are referencing **landingzone.key** of our **testconfig1** deployment.

Putting that information together, we are declaring that we want this storage account deployed into a resource group managed by a remote Terraform deployment (**test_config1**).  Because we declared a tfstate dependency on **test_config1**, rover & the aztfmod module will work together to bring in all of the resource information contained within the **test_config1** tfstate file.  The resource information will be stored in local variables within our current deployment context.  The module can then reference the properties it needs from the **testconfig1** deployment resource group identified by the key **evh_examples** like this:
`local.combined_objects_resource_groups["test_config1"]["evh_examples"].PROPERTY_NAME`

This configuration can be deployed by executing the following rover command:
```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_solution/ -a apply \
            -tfstate test_config2_storage.tfstate \
            -level level1 \
            -parallelism=30 \
            -var-folder /tf/caf/configuration/level1/testconfig2 \
            --environment contoso
```

When complete, you will find the storage account you defined in the **testconfig2** deployment located within the resource group created by deployment **testconfig1**.

---
## Wrap up ##
---

We've seen how to leverage the CAF Terraform Supermodule with the Rover toolset to deploy modularized Terraform configurations with interdependencies.

If you wish to remove the resources created by this walkthrough, you should execute the following to remove **testconfig2**:

```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_solution/ -a destroy \
            -tfstate test_config2_storage.tfstate \
            -level level1 \
            -parallelism=30 \
            -var-folder /tf/caf/configuration/level1/testconfig2 \
            --environment contoso
```

To remove **testconfig1**:

```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_solution/ -a destroy \
            -tfstate test_config1.tfstate \
            -level level1 \
            -parallelism=30 \
            -var-folder /tf/caf/configuration/level1/testconfig1 \
            --environment contoso
```

To remove **foundations**:

```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_solution -a destroy \
            -var-folder /tf/caf/landingzones/caf_solution/scenario/foundations/100-passthrough \
            -tfstate caf_foundations.tfstate \
            -level level1 \
            -parallelism=30 \
            --environment contoso
```

If you intend to follow the more advanced walkthroughs in this tutorial, it is recommended that you leave the launchpad deployment in place in order to save time.  Otherwise, to remove the **launchpad**:

```bash
/tf/rover/rover.sh -lz /tf/caf/landingzones/caf_launchpad -a apply \
            -var-folder /tf/caf/landingzones/caf_launchpad/scenario/100 \
            -level level0 \
            -launchpad \
            -parallelism=30 \
            --environment contoso \
            '-var prefix=cont'
```

You've reached the end of the custom deployment walkthrough.  You may stop here or proceed to the [CAF Platform Landing Zones Walkthrough](L_platform_deploy.md)
