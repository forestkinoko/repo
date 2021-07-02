# Eventhub

## Create Eventhub
```
# Set  variables.
my_resource_group=RESOURCEGROUPNAME
my_namespace=EVENTHUBNAMESPACE

# Create Namespace
az eventhubs namespace create --resource-group $my_resource_group --name $my_namespace --location japaneast --sku Basic --maximum-throughput-units 1

# Create Eventhub, for Syslog
az eventhubs eventhub create --resource-group $my_resource_group --namespace-name $my_namespace --name syslog --message-retention 1 --partition-count 4
```

## Linux Diagnostic Extension (LAD) 4.0 Install
[Document](https://docs.microsoft.com/ja-jp/azure/virtual-machines/extensions/diagnostics-linux?tabs=azcli)
```
# Set your Azure VM diagnostic variables.
my_resource_group=RESOURCEGROUPNAME
my_linux_vm=VMNAME
my_diagnostic_storage_account=STORAGEACCOUNTNAME
my_namespace=EVENTHUBNAMESPACE

# Build the VM resource ID. Replace the storage account name and resource ID in the public settings.
my_vm_resource_id=$(az vm show -g $my_resource_group -n $my_linux_vm --query "id" -o tsv)
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT__#$my_diagnostic_storage_account#g" public_settings_metric_none.json
sed -i "s#__VM_RESOURCE_ID__#$my_vm_resource_id#g" public_settings_metric_none.json
cat public_settings_metric_none.json

# Build the protected settings (storage account SAS token).
my_diagnostic_storage_account_sastoken=$(az storage account generate-sas --account-name $my_diagnostic_storage_account --expiry 2037-12-31T23:59:00Z --permissions wlacu --resource-types co --services bt -o tsv)
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT__#$my_diagnostic_storage_account#g" protected_settings.json
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT_SASTOKEN__#$my_diagnostic_storage_account_sastoken#g" protected_settings.json

# Build the protected settings (eventhub SAS url)
eventhub_saskey=$(az eventhubs namespace authorization-rule keys list --resource-group rin-rg-prd --namespace-name rinnamespace --name RootManageSharedAccessKey -o tsv | awk '{print $5}')
echo "get_sas_token '$my_namespace.servicebus.windows.net/syslog' 'RootManageSharedAccessKey' '$eventhub_saskey'" >> get_sas_token.sh
eventhub_sastoken=$(bash get_sas_token.sh | awk '{print $2}')
eventhub_sasurl=$(echo "https://$my_namespace.servicebus.windows.net/syslog?$eventhub_sastoken")
sed -i "s#__SAS_URL__#$eventhub_sasurl#g" protected_settings.json
cat protected_settings.json

# install and enable the extension.
az vm extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 4.0 --resource-group $my_resource_group --vm-name $my_linux_vm --protected-settings protected_settings.json --settings public_settings_metric_none.json
```
