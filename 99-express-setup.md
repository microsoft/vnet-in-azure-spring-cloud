# Express Setup (WIP)

This document will guide you to deploy Spring microservices in your own Azure Virtual Network using Azure Spring Cloud and MySQL, and expose those microservices on Internet by integrating with Azure Application Gateway and Azure Firewall.

## What will you experience
You will:
- Build existing Spring microservices applications
- Provision an Azure Spring Cloud service instance in your own Azure Virtual Network
- Deploy applications to Azure Spring Cloud
- Bind applications to Azure Database for MySQL in your own Azure Virtual Network
- Open the application on Jumpbox machine in vnet

## What you will need

In order to deploy a Java app to cloud, you need 
an Azure subscription. If you do not already have an Azure 
subscription, you can activate your 
[MSDN subscriber benefits](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) 
or sign up for a 
[free Azure account]((https://azure.microsoft.com/free/)).

Azure Portal link: https://ms.portal.azure.com/?AppPlatformExtension=vnet

In addition, you will need the following:

| [Azure CLI version 2.0.67 or higher](https://docs.microsoft.com/cli/azure/install-azure-cli?view=azure-cli-latest) 
| [Java 8](https://www.azul.com/downloads/azure-only/zulu/?version=java-8-lts&architecture=x86-64-bit&package=jdk) 
| [Maven](https://maven.apache.org/download.cgi) 
| [MySQL CLI](https://dev.mysql.com/downloads/shell/)
| [Git](https://git-scm.com/)

## Install the Azure CLI extension

Install the Azure Spring Cloud extension for the Azure CLI using the following command

```bash
az extension add -s https://azureclitemp.blob.core.windows.net/spring-cloud/spring_cloud-0.3.0_preview_vnet-py2.py3-none-any.whl
```

If you have already installed, please remove it first

```bash
az extension remove -n spring-cloud
```

## Clone and build the repo

### Create a new folder and clone the sample app repository to your Azure Cloud account  

```bash
mkdir source-code
git clone https://github.com/azure-samples/spring-petclinic-microservices
```

### Change directory and build the project

```bash
cd spring-petclinic-microservices
mvn clean package -DskipTests -Denv=cloud
```
This will take a few minutes.

## Create a resource group and a virtual network

### Prepare your environment for deployments

Download [setup-env-variables-azure-template.sh](setup-env-variables-azure-template.sh), and create a bash script with environment variables by making a copy of the supplied template:
```bash
cp setup-env-variables-azure-template.sh setup-env-variables-azure.sh
```

Open `setup-env-variables-azure.sh` and enter the following information:

```bash
export SUBSCRIPTION=subscription-id # customize this
export RESOURCE_GROUP=resource-group-name # customize this
...
export SPRING_CLOUD_SERVICE=azure-spring-cloud-name # customize this
...
export MYSQL_SERVER_NAME=mysql-servername # customize this
...
export MYSQL_SERVER_ADMIN_NAME=admin-name # customize this
...
export MYSQL_SERVER_ADMIN_PASSWORD=SuperS3cr3t # customize this
...
```

Then, set the environment:
```bash
source setup-env-variables-azure.sh
```

### Login to Azure 
Login to the Azure CLI and choose your active subscription. Be sure to choose the active subscription that is whitelisted for Azure Spring Cloud

```bash
az login
az account list -o table
az account set --subscription ${SUBSCRIPTION}
```

### Create a resource group and a virtual network

Set default values using the following commands:

```bash
az configure --defaults \
    group=${RESOURCE_GROUP} \
    location=${REGION} \
    spring-cloud=${SPRING_CLOUD_SERVICE}
```

Create a resource group to contain your Azure Virtual Network and Azure Spring Cloud service.

```bash
az group create --name ${RESOURCE_GROUP} --location ${REGION}
```

Create a virtual network.

```bash
az network vnet create \
    --name ${VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP} \
    --address-prefixes 10.1.0.0/16
```

Create two subnets, one for Azure Spring Cloud service runtime and another for your Spring Microservices.

```bash
az network vnet subnet create \
    --name service-runtime-subnet \
    --vnet-name ${VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP} \
    --address-prefixes 10.1.0.0/24

az network vnet subnet create \
    --name apps-subnet \
    --vnet-name ${VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP} \
    --address-prefixes 10.1.1.0/24
```

Grant Azure Spring Cloud service permission to the virtual network

```bash
VIRTUAL_NETWORK_RESOURCE_ID=`az network vnet show \
    --name ${VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP} \
    --query "id" \
    --output tsv`

az role assignment create \
    --role "Owner" \
    --scope ${VIRTUAL_NETWORK_RESOURCE_ID} \
    --assignee e8de9221-a19c-4c81-b814-fd37c6caf9d2
```

### Create Azure Spring Cloud service instance in virtual network

Create an instance of Azure Spring Cloud in above created virtual network.

```bash
az spring-cloud create --name ${SPRING_CLOUD_SERVICE} \
    --resource-group ${RESOURCE_GROUP} \
    --location ${REGION} \
    --vnet ${VIRTUAL_NETWORK} \
    --service-runtime-subnet service-runtime-subnet \
    --app-subnet apps-subnet
```

### Load Spring Cloud Config Server

Go to the root of the sample app project, and use the `application.yml` to load configuration into the Config Server in Azure Spring Cloud.

```bash
az spring-cloud config-server set \
    --config-file application.yml \
    --name ${SPRING_CLOUD_SERVICE}
```

## Create microservice applications

Create 5 microservice apps.

```bash
az spring-cloud app create --name ${API_GATEWAY} --instance-count 1 --is-public true \
    --memory 2 \
    --jvm-options='-Xms2048m -Xmx2048m'

az spring-cloud app create --name ${ADMIN_SERVER} --instance-count 1 --is-public true \
    --memory 2 \
    --jvm-options='-Xms2048m -Xmx2048m'

az spring-cloud app create --name ${CUSTOMERS_SERVICE} --instance-count 1 \
    --memory 2 \
    --jvm-options='-Xms2048m -Xmx2048m'

az spring-cloud app create --name ${VETS_SERVICE} --instance-count 1 \
    --memory 2 \
    --jvm-options='-Xms2048m -Xmx2048m'

az spring-cloud app create --name ${VISITS_SERVICE} --instance-count 1 \
    --memory 2 \
    --jvm-options='-Xms2048m -Xmx2048m'
```

## Create MySQL Database

Create a MySQL database in Azure Database for MySQL.

```bash
// create mysql server
az mysql server create --resource-group ${RESOURCE_GROUP} \
    --name ${MYSQL_SERVER_NAME}  --location ${REGION} \
    --admin-user ${MYSQL_SERVER_ADMIN_NAME} \
    --admin-password ${MYSQL_SERVER_ADMIN_PASSWORD} \
    --sku-name GP_Gen5_2 \
    --ssl-enforcement Disabled \
    --version 5.7

// allow access from your dev machine for testing
az mysql server firewall-rule create --name devMachine \
    --server ${MYSQL_SERVER_NAME} \
    --resource-group ${RESOURCE_GROUP} \
    --start-ip-address 0.0.0.0 \
    --end-ip-address 255.255.255.255

// increase connection timeout
az mysql server configuration set --name wait_timeout \
    --resource-group ${RESOURCE_GROUP} \
    --server ${MYSQL_SERVER_NAME} --value 2147483

// SUBSTITUTE values
mysql -u ${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
    -h ${MYSQL_SERVER_FULL_NAME} -P 3306 -p

Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 64379
Server version: 5.6.39.0 MySQL Community Server (GPL)

Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql> CREATE DATABASE petclinic;
Query OK, 1 row affected (0.10 sec)

mysql> CREATE USER 'root' IDENTIFIED BY 'petclinic';
Query OK, 0 rows affected (0.11 sec)

mysql> GRANT ALL PRIVILEGES ON petclinic.* TO 'root';
Query OK, 0 rows affected (1.29 sec)

mysql> CALL mysql.az_load_timezone();
Query OK, 3179 rows affected, 1 warning (6.34 sec)

mysql> SELECT name FROM mysql.time_zone_name;
...

mysql> quit
Bye


az mysql server configuration set --name time_zone \
    --resource-group ${RESOURCE_GROUP} \
    --server ${MYSQL_SERVER_NAME} --value "US/Pacific"
```

Create subnet for the MySQL database.

```bash
az network vnet subnet create \
    --name mysql-subnet \
    --vnet-name ${VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP} \
    --address-prefixes 10.1.2.0/24
```

Disable subnet private endpoint policies

```bash
az network vnet subnet update \
    --name mysql-subnet \
    --vnet-name ${VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP} \
    --disable-private-endpoint-network-policies true
```

Disable public access for MySQL database and add private link on Azure Portal, followed by [Configure Private Link for Azure Database for MySQL](https://docs.microsoft.com/en-us/azure/mysql/concepts-data-access-security-private-link#configure-private-link-for-azure-database-for-mysql).

## Deploy applications and set environment variables

Deploy microservice applications to Azure.

```bash
az spring-cloud app deploy --name ${API_GATEWAY} \
    --jar-path ${API_GATEWAY_JAR} \
    --jvm-options='-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql'

ADMIN_SERVER_URL=`az spring-cloud app show --name ${ADMIN_SERVER} --query "properties.url" -o tsv`
az spring-cloud app deploy --name ${ADMIN_SERVER} \
    --jar-path ${ADMIN_SERVER_JAR} \
    --jvm-options="-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql -Dspring.boot.admin.ui.public-url=${ADMIN_SERVER_URL}"


az spring-cloud app deploy --name ${CUSTOMERS_SERVICE} \
    --jar-path ${CUSTOMERS_SERVICE_JAR} \
    --jvm-options='-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql' \
    --env MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_FULL_NAME} \
            MYSQL_DATABASE_NAME=${MYSQL_DATABASE_NAME} \
            MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
            MYSQL_SERVER_ADMIN_PASSWORD=${MYSQL_SERVER_ADMIN_PASSWORD}


az spring-cloud app deploy --name ${VETS_SERVICE} \
    --jar-path ${VETS_SERVICE_JAR} \
    --jvm-options='-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql' \
    --env MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_FULL_NAME} \
            MYSQL_DATABASE_NAME=${MYSQL_DATABASE_NAME} \
            MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
            MYSQL_SERVER_ADMIN_PASSWORD=${MYSQL_SERVER_ADMIN_PASSWORD}
            

az spring-cloud app deploy --name ${VISITS_SERVICE} \
    --jar-path ${VISITS_SERVICE_JAR} \
    --jvm-options='-Xms2048m -Xmx2048m -Dspring.profiles.active=mysql' \
    --env MYSQL_SERVER_FULL_NAME=${MYSQL_SERVER_FULL_NAME} \
            MYSQL_DATABASE_NAME=${MYSQL_DATABASE_NAME} \
            MYSQL_SERVER_ADMIN_LOGIN_NAME=${MYSQL_SERVER_ADMIN_LOGIN_NAME} \
            MYSQL_SERVER_ADMIN_PASSWORD=${MYSQL_SERVER_ADMIN_PASSWORD}
```

## Create a Windows Jumpbox machine to access the application

Create subnet for the Jumpbox machine

```bash
az network vnet subnet create \
    --name jumpbox-subnet \
    --vnet-name ${VIRTUAL_NETWORK} \
    --resource-group ${RESOURCE_GROUP} \
    --address-prefixes 10.1.4.0/24
```

Create a Windows VM on Azure Portal join above VNet, and then you can access the applications on the jumpbox machine.