
# Deploy the Platform Landing Zones using the provided templates
---

If you've read through the platform-starter walkthough, you'll have noticed that we have not yet departed from the instructions it provides.  In fact, if you want to deploy all of the resources used by the Platform Landing Zone subscriptions, you could do that at this time by executing the deployment bash script located at `/tf/caf/landingzones/templates/platform/deploy_platform.sh`.  However, there are multiple drawbacks to starting with this approach, including:
- Requires privileged access to both Azure and Azure AD
- It is possible to deploy into a MSDN subscription; however, when fully deployed, it incurs significant costs
- It takes a significant amount of time to walk through and execute the many nested deployments required for this scenario
- The deployment is largely pre-configured, and offers few opportunities to increase your understanding of how rover works or how templates, definition files and configuration files are related.

>  In my opinion, it is this last bullet point that is most significant.  It required a huge time investment to reverse engineer how the scripts and templates interact to produce the final product.

If you want to continue with this approach, refer to the walkthrough which begins here: https://aztfmod.github.io/documentation/docs/azure-landing-zones/landingzones/platform/single%20reuse/elsz-single-reuse
