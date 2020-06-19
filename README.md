## Steps to do:

### Prerequiste:

#### Steps to setup Mysql 

1. Create resource group
2. Create Mysql server on Azure (Azure database for mysql)
Reference for mysql :
https://docs.microsoft.com/en-us/azure/developer/java/spring-framework/configure-spring-data-jpa-with-azure-mysql

3. Allow your client address in mysql firewall
and set "Allow access to Azure services" as YES

4. Create mysql database:
mysql --host testpacker.mysql.database.azure.com --user azureuser@testpacker -p

CREATE DATABASE packerdb;
SHOW DATABASES;
=========================

#### Steps to package application
1. Clone Git repo
2. Edit mysql database connection string in application.properties under src/main/resources in application
For Reference, check: https://github.com/ashikansal/mysql-spring-boot-todo.git

3. Package application
mvn package

4. Test application on local:
java -jar target/TodoDemo-0.0.1-SNAPSHOT.war

5. Check localhost:8080 
application must be running

#### Steps to prepare Packer Image

1. Create a service principal:

az ad sp create-for-rbac -n sp_for_packer --query "{ client_id: appId, client_secret: password, tenant_id: tenant }"

Output like this:
{
  "client_id": "deca58c5-6d3e-47cd-910c-1b62d057a71f",
  "client_secret": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx",
  "tenant_id": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
}

2. Clone this repo

3. Replace client_id, client_secret, tenant_id in your packer.json with the above generated values.
Replace subscription_id with the one of your azure account.

4. Create Resource group beforehand (already done while creating mysql server) and pass same name in "managed_image_resource_group_name" and update "location" accordingly

5. Image name can be fetched from below command:
az vm image list --output table

6. Install packer in your machine

7. Replace your war file with the war file in repo so that database connection string is correct

8. Run packer build from inside repo after editing variables accordingly
packer build linux_packer.json

9. Create VM
az vm create \
       --resource-group packerTestRG \
       --name TestLinuxVM1 \
       --image PackerLinuxJavaImage \
       --admin-username azureuser \
       --admin-password abcde@123456 \
       --authentication-type password \
       --generate-ssh-keys

10. Open port 8080
az vm open-port \
    --resource-group packerTestRG \
    --name TestLinuxVM1 \
    --port 8080
           
11. Check if application running:
public_ip:8080
