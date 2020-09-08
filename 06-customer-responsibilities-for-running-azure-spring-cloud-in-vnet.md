# Customer Responsibilities for Running Azure Spring Cloud in VNET

- test

## **Don't do** list for customers

- Please **don't** modify resource groups created and owned by Azure Spring Cloud.
  - By default, those resource groups are named as *azure-spring-cloud-service-runtime_[SERVICE-INSTANCE-NAME]_[REGION]* and *azure-spring-cloud-app_[SERVICE-INSTANCE-NAME]_[REGION]*.
- Please **don't** modify subnets used by Azure Spring Cloud.
- Please **don't** create more than one Azure Spring Cloud service instance in the same subnet.
- When use Firewall to control traffic, please **don't** block the following egress traffic to Azure Spring Cloud components for operating, maintaining and supporting the service instance.

  - Azure Spring Cloud required network rules

    | Destination Endpoint | Port | Use |
    |------|------|------|
    | *:1194 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureCloud:1194 | UDP:1194 | Underlying Kubernetes Cluster management. |
    | *:443 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureCloud:443 | TCP:443 | Azure Spring Cloud service management. |
    | *:9000 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureCloud:9000 | TCP:9000 | Underlying Kubernetes Cluster management. |
    | *:123 *Or* ntp.ubuntu.com:123 | UDP:123 | NTP time synchronization on Linux nodes. |
    | *.azure.io:443 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureContainerRegistry:443 | TCP:443 | Azure Container Registry. |
    | *.file.core.windows.net:445 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - Storage:445 | TCP:445 | Azure File Storage. |

  - Azure Spring Cloud required FQDN / application rules
    - Azure Firewall provides a FQDN Tag "AzureKubernetesService" to simplify all following configurations.

    | Destination FQDN | Port | Use |
    |------|------|------|
    | *.azmk8s.io | HTTPS:443 | Underlying Kubernetes Cluster management. |
    | <i>mcr.microsoft.com</i> | HTTPS:443 | Microsoft Container Registry (MCR). |
    | *.cdn.mscr.io | HTTPS:443 | MCR storage backed by the Azure CDN. |
    | *.data.mcr.microsoft.com | HTTPS:443 | MCR storage backed by the Azure CDN. |
    | <i>management.azure.com</i> | HTTPS:443 | Underlying Kubernetes Cluster management. ​|
    | <i>login.microsoftonline.com</i> | HTTPS:443 | Azure Active Directory authentication.​ |
    |<i>packages.microsoft.com</i>    | HTTPS:443 | Microsoft packages repository. |
    | <i>acs-mirror.azureedge.net</i> | HTTPS:443 | Repository required to install required binaries like kubenet and Azure CNI.​ |