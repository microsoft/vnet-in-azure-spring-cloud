# Customer Responsibilities for Running Azure Spring Cloud in VNET

## **Don't do** list for customers

- Please **don't** modify resource groups created and owned by Azure Spring Cloud.
  - By default, those resource groups are named as *azure-spring-cloud-service-runtime_[SERVICE-INSTANCE-NAME]_[REGION]* and *azure-spring-cloud-app_[SERVICE-INSTANCE-NAME]_[REGION]*.
- Please **don't** modify subnets used by Azure Spring Cloud.
- Please **don't** create more than one Azure Spring Cloud service instance in the same subnet.
- When use Firewall to control traffic, please **don't** block the following egress traffic to Azure Spring Cloud components for operating, maintaining and supporting the service instance.

    |FQDN                             |Port                              |Use                                        |
    |---------------------------------|----------------------------------|-------------------------------------------|
    |*.blob.core.windows.net          |HTTPS:443                         |Azure Spring Cloud storage management.     ​|
    |*.file.core.windows.net          |HTTPS:443                         |Azure Spring Cloud storage management.     ​|
    |*.azurecr.io                     |HTTPS:443                         |Azure Spring Cloud storage management.     ​|
    |*.metrics.nsatc.net​              |HTTPS:443                         |Microsoft internal logs and metrics server.​|
    |*.hcp.<location>.azmk8s.io       |HTTPS:443,TCP:22,TCP:9000,UDP:1194|Underlying Kubernetes Cluster API Server.  |
    |*.tun.<location>.azmk8s.io       |HTTPS:443,TCP:22,TCP:9000,UDP:1194|Underlying Kubernetes Cluster API Server.  |
    |*.cdn.mscr.io                    |HTTPS:443                         |MCR storage backed by the Azure CDN.       ​|
    |<i>mcr.microsoft.com</i>         |HTTPS:443                         |Microsoft Container Registry (MCR).        ​|
    |*.data.mcr.microsoft.com         |HTTPS:443                         |MCR storage backed by the Azure CDN.       |
    |<i>management.azure.com</i>      |HTTPS:443                         |Underlying Kubernetes Cluster management.  ​|
    |<i>login.microsoftonline.com</i> |HTTPS:443                         |Azure Active Directory authentication.​     |
    |<i>ntp.ubuntu.com</i>            |UDP:123                           |NTP time synchronization on Linux nodes.​   |
    |<i>packages.microsoft.com</i>    |HTTPS:443                         |Microsoft packages repository.​             |
    |<i>acs-mirror.azureedge.net</i>  |HTTPS:443                         |Repository required to install required binaries like kubenet and Azure CNI.​|