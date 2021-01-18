# Troubleshooting Azure Spring Cloud in VNET

## I encountered a problem with creating an Azure Spring Cloud service instance

First of all, please ensure you grant sufficient permission to the virtual network resource your service instance deployed in, followed by [Grant Azure Spring Cloud service permission to the virtual network](01-deploy-azure-spring-cloud-in-your-vnet.md#grant-azure-spring-cloud-service-permission-to-the-virtual-network).

If the issue still occur, when you set up an Azure Spring Cloud service instance by using the Azure portal, Azure Spring Cloud performs the validation for you.

But if you try to set up the Azure Spring Cloud service instance by using the [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/get-started-with-azure-cli), verify that:

- The subscription is active.
- The location is supported by Azure Spring Cloud.
- The resource group for the instance is already created.
- The resource name conforms to the naming rule. It must contain only lowercase letters, numbers, and hyphens. The first character must be a letter. The last character must be a letter or number. The value must contain from 2 to 32 characters.

If you want to set up the Azure Spring Cloud service instance by using the Resource Manager template, first refer to [Understand the structure and syntax of Azure Resource Manager templates](https://docs.microsoft.com/en-us/azure/azure-resource-manager/resource-group-authoring-templates).

### Common creation issues

| Error Message | How to fix |
|------|------|
| Resources created by Azure Spring Cloud were disallowed by policy. | Network resources will be created when deploy Azure Spring Cloud in your own virtual network. Please check whether you have [Azure Policy](https://docs.microsoft.com/en-us/azure/governance/policy/overview) defined to block those creation. Resources failed to be created can be found in error message. |
| Provided subnets have associated with route tables, please disassociate them. | Currently it is not supported to deploy Azure Spring Cloud in subnet associated with existing route tables, please dissociate them and try again. |
| Required traffic is not whitelisted. | Please refer to [Customer Responsibilities for Running Azure Spring Cloud in VNET](06-customer-responsibilities-for-running-azure-spring-cloud-in-vnet.md) to ensure required traffic is whitelisted. |

## My application can't be registered

The problem happens if your virtual network is configured with a custom DNS settings. In that case, private DNS zone used by Azure Spring Cloud would be ineffective. Please add the Azure DNS IP 168.63.129.16 as the upstream DNS server in the custom DNS server.

## Other issues

Please reference [Troubleshoot common Azure Spring Cloud issues](https://docs.microsoft.com/en-us/azure/spring-cloud/spring-cloud-troubleshoot).