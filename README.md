# Back up an Azure virtual machine
This is to setup and implement an Azure Backup for both Windows and Linux workloads by using a combination of the Azure CLI and the Azure portal.
Azure Backup will be enable from the portal, from the Azure CLI, or by using PowerShell commands.
First is creation of VM, setting up the backup, and starting the backup.

# Create a backup for Azure virtual machines

## Set up the environment
1. Sign in to the Azure portal, and open Azure Cloud Shell.
 ![image](https://github.com/eleasel/AzureBackup/assets/101367394/c2db5c1e-945f-41a6-bfce-df2393804c49)

.

## Create a resource group to contain all the resources for this exercise.

RGROUP=$(az group create --name vmbackups --location westus2 --output tsv --query name)

## Use Cloud Shell to create the NorthwindInternal virtual network and the NorthwindInternal1 subnet.

az network vnet create \
    --resource-group $RGROUP \
    --name NorthwindInternal \
    --address-prefixes 10.0.0.0/16 \
    --subnet-name NorthwindInternal1 \
    --subnet-prefixes 10.0.0.0/24

# Create a Windows virtual machine by using the Azure CLI
Create the NW-APP01 virtual machine by running the following command. Replace <password> with a password of your choice, enclosed in double quotes. For example, --admin-password "PassWord123!".

az vm create \
    --resource-group $RGROUP \
    --name NW-APP01 \
    --size Standard_DS1_v2 \
    --public-ip-sku Standard \
    --vnet-name NorthwindInternal \
    --subnet NorthwindInternal1 \
    --image Win2016Datacenter \
    --admin-username admin123 \
    --no-wait \
    --admin-password <password>

# Create a Linux virtual machine by using the Azure CLI
Create the NW-RHEL01 virtual machine by running the following command.

az vm create \
    --resource-group $RGROUP \
    --name NW-RHEL01 \
    --size Standard_DS1_v2 \
    --image RedHat:RHEL:7-RAW:latest \
    --authentication-type ssh \
    --generate-ssh-keys \
    --vnet-name NorthwindInternal \
    --subnet NorthwindInternal1
The command can take a few minutes to complete. Wait for it to finish before moving on to the next step.

# Enable backup for a virtual machine by using the Azure portal
1. In the Azure portal, search for and select Virtual machines.
![image](https://github.com/eleasel/AzureBackup/assets/101367394/8b6a0da4-ea6d-4355-bc72-3ed8f118dd0a)

The Virtual machines pane appears.


2. From the list, select the NW-RHEL01 virtual machine that you created.

![image](https://github.com/eleasel/AzureBackup/assets/101367394/6003a886-c397-4951-bdca-459088446636)

The NW-RHEL01 virtual machine pane appears.

3. In the middle menu pane, scroll down to Operations, and select Backup. The Backup pane for the NW-RHEL01 virtual machine appears.

4. Under the Summary section, ensure the following information exists for creating a backup.
Recovery services vault: azure-backup for the name.
Backup policy: DailyPolicy-xxxxxxxx, which creates a daily backup at 12:00 PM UTC with a retention range of 180 days.
![image](https://github.com/eleasel/AzureBackup/assets/101367394/e083ca65-cd77-4f75-ad5d-fdf4155914e3)

5. To perform the first backup for this server, in the top menu bar, select Backup now.

![image](https://github.com/eleasel/AzureBackup/assets/101367394/228cd239-84c7-437b-b804-909d0d5a9281)

The Backup Now pane for NW-RHEL01 appears.

6. Select OK.

# Enable a backup by using the Azure CLI
1. Using Cloud Shell, enable a backup for the NW-APP01 virtual machine.

az backup protection enable-for-vm \
    --resource-group vmbackups \
    --vault-name azure-backup \
    --vm NW-APP01 \
    --policy-name DefaultPolicy

2. Monitor the progress of the setup using the Azure CLI.

az backup job list \
    --resource-group vmbackups \
    --vault-name azure-backup \
    --output table
Keep running the preceding command until you see that ConfigureBackup has finished.

Name                                  Operation        Status      Item Name    Start Time UTC                    Duration
------------------------------------  ---------------  ----------  -----------  --------------------------------  --------------
a3df79b4-be4f-4cc9-8b2c-a5ead44a6a12  ConfigureBackup  Completed   NW-APP01     2019-08-01T06:19:12.101048+00:00  0:00:31.305975
5e1531a9-8b3d-4983-a642-86ee982f7036  Backup           InProgress  NW-RHEL01    2019-08-01T06:18:35.955118+00:00  0:01:22.734182
860d4dca-9603-4a4e-9f3b-93f242a0a64d  ConfigureBackup  Completed   NW-RHEL01    2019-08-01T06:13:33.860598+00:00  0:00:31.256773    

3. Do an initial backup of the virtual machine, instead of waiting for the schedule to run it.

az backup protection backup-now \
    --resource-group vmbackups \
    --vault-name azure-backup \
    --container-name NW-APP01 \
    --item-name NW-APP01 \
    --retain-until 18-10-2030 \
    --backup-management-type AzureIaasVM
There's no need to wait for the backup to finish, because you'll see how to monitor the progress in the portal next.

# Monitor backups in the portal
## View the status of a backup for a single virtual machine
1. On the Azure portal menu or from the Home page, select All resources.
2. Enter Virtual machines in the search field at the top of the page and select Virtual machines from the results.
3. Select the NW-APP01 virtual machine. The NW-APP01 virtual machine pane appears.
4. In the middle menu pane, scroll to Operations, and select Backup. The Backup pane for the NW-APP01 virtual machine appears.

Under the Backup status section, the Last backup status field displays the current status of the backup.

![image](https://github.com/eleasel/AzureBackup/assets/101367394/91893d40-2522-4321-9be7-b62bc1fe8ee3)

## View the status of backups in the Recovery Services vault
1. On the Azure portal menu or from the Home page, select All resources.
2. Sort the list by Type, and then select the azure-backup Recovery Services vault. The Azure-backup recovery services vault pane appears.
3. On the Overview pane, select the interior Backup tab to display a summary of all the backup items, the storage being used, and the current status of any backup jobs.

![image](https://github.com/eleasel/AzureBackup/assets/101367394/cd17c786-7c98-4e94-b2e6-9f01ff97b78f)

