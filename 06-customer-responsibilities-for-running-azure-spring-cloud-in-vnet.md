# Customer Responsibilities for Running Azure Spring Cloud in VNET

## Background

When Azure Spring Cloud is deployed in your virtual network, it has **outbound** dependencies on services outside of the virtual network. For management and operational purposes, Azure Spring Cloud need to access certain ports and fully qualified domain names (FQDNs). These endpoints are required to communicate with Azure Spring Cloud management plane, and to download and install core Kubernetes cluster components and security updates.

By default, Azure Spring Cloud have unrestricted outbound (egress) internet access. This level of network access allows Applications you run to access external resources as needed. If you wish to restrict egress traffic, a limited number of ports and addresses must be accessible for maintenance tasks. The simplest solution to securing outbound addresses lies in use of a firewall device that can control outbound traffic based on domain names. Azure Firewall, for example, can restrict outbound HTTP and HTTPS traffic based on the FQDN of the destination. You can also configure your preferred firewall and security rules to allow these required ports and addresses.

## **Don't do** list for customers

- Please **don't** modify resource groups created and owned by Azure Spring Cloud.
  - By default, those resource groups are named as *ap-svc-rt_[SERVICE-INSTANCE-NAME]_[REGION]* and *ap-app_[SERVICE-INSTANCE-NAME]_[REGION]*.
- Please **don't** modify subnets used by Azure Spring Cloud.
- Please **don't** create more than one Azure Spring Cloud service instance in the same subnet.
- When use Firewall to control traffic, please **don't** block the following egress traffic to Azure Spring Cloud components for operating, maintaining and supporting the service instance.

  - **Azure Spring Cloud required network rules**

    | Destination Endpoint | Port | Use | Note |
    |------|------|------|------|
    | *:1194 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureCloud:1194 | UDP:1194 | Underlying Kubernetes Cluster management. ||
    | *:443 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureCloud:443 *Or* ServiceInstanceRequiredTraffics:443 (only known after creation) | TCP:443 | Azure Spring Cloud service management. | Information of service instance "requiredTraffics" could be known in resource payload, under "networkProfile" section.  |
    | *:9000 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureCloud:9000 | TCP:9000 | Underlying Kubernetes Cluster management. ||
    | *:123 *Or* ntp.ubuntu.com:123 | UDP:123 | NTP time synchronization on Linux nodes. ||
    | *.azure.io:443 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - AzureContainerRegistry:443 | TCP:443 | Azure Container Registry. | Can be replaced by enabling *Azure Container Registry* [service endpoint in virtual network](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview). |
    | *.core.windows.net:443 and *.core.windows.net:445 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - Storage:443 and Storage:445 | TCP:443, TCP:445 | Azure File Storage. | Can be replaced by enabling *Azure Storage* [service endpoint in virtual network](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview). |
    | *.servicebus.windows.net:443 *Or* [ServiceTag](https://docs.microsoft.com/en-us/azure/virtual-network/service-tags-overview#available-service-tags) - EventHub:443 | TCP:443 | Azure Event Hub. | Can be replaced by enabling *Azure Event Hubs* [service endpoint in virtual network](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-network-service-endpoints-overview). |

  - **Azure Spring Cloud required FQDN / application rules**
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
    | <i>mscrl.microsoft.com</i> | HTTPS:80 | Required Microsoft Certificate Chain Paths.​ |
    | <i>crl.microsoft.com</i> | HTTPS:80 | Required Microsoft Certificate Chain Paths.​ ​ |
    | <i>crl3.digicert.com</i> | HTTPS:80 | 3rd Party SSL Certificate Chain Paths.​  |

  - Azure Spring Cloud optional FQDN / application rules
    - Azure Firewall provides a FQDN Tag "AzureKubernetesService" to simplify all following configurations.
    - Third party APM (Application Performance Management) solutions.
      | APM provider                       | Network Details                                              |
      | ---------------------------------- | ------------------------------------------------------------ |
      | [New Relic](https://newrelic.com/) | [APM Agents](https://docs.newrelic.com/docs/using-new-relic/cross-product-functions/install-configure/networks/#agents) |
