# Eventhub

## Linux Diagnostic Extension (LAD) 4.0 Install
[Document](https://docs.microsoft.com/ja-jp/azure/virtual-machines/extensions/diagnostics-linux?tabs=azcli)
```
# Set your Azure VM diagnostic variables.
my_resource_group=RESOURCEGROUPNAME
my_linux_vm=VMNAME
my_diagnostic_storage_account=STORAGEACCOUNTNAME

# Build the VM resource ID. Replace the storage account name and resource ID in the public settings.
my_vm_resource_id=$(az vm show -g $my_resource_group -n $my_linux_vm --query "id" -o tsv)
sed -i "s#__DIAGNOSTIC_STORAGE_ACCOUNT__#$my_diagnostic_storage_account#g" public_settings_metric_none.json
sed -i "s#__VM_RESOURCE_ID__#$my_vm_resource_id#g" public_settings_metric_none.json

# install and enable the extension.
az vm extension set --publisher Microsoft.Azure.Diagnostics --name LinuxDiagnostic --version 4.0 --resource-group $my_resource_group --vm-name $my_linux_vm --protected-settings protected_settings.json --settings public_settings_metric_none.json
```
