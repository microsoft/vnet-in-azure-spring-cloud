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

## My application can't be registered

The problem happens if your virtual network is configured with a custom DNS settings. In that case, private DNS zone used by Azure Spring Cloud would be ineffective. Please add the Azure DNS IP 168.63.129.16 as the upstream DNS server in the custom DNS server.

## Other issues

Please reference [Troubleshoot common Azure Spring Cloud issues](https://docs.microsoft.com/en-us/azure/spring-cloud/spring-cloud-troubleshoot).