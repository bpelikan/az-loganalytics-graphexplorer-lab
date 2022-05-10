# Resource Graph

* `resources` - tabela przechowująca informacje o obecnych zasobach
* `resourcechanges` - tabela przechowująca zmiany na zasobach ([Get resource changes](https://docs.microsoft.com/en-us/azure/governance/resource-graph/how-to/get-resource-changes))

#### 1. Liczebność zasobów według danego typu
```kql
Resources
| summarize resource_count=count() by tostring(resource_type=type)
| order by resource_count
```

<details>
  <summary><b><i>Results</i></b></summary>

![Screen](./img/20220509220001.jpg "Screen")
</details>


#### 2. Największa liczba zmian w ciągu 1 minuty
```kql
resourcechanges 
| extend changeTime=format_datetime(todatetime(properties.changeAttributes.timestamp), 'yy-MM-dd HH:mm')
| summarize changes_count=count() by tostring(changeTime)
| order by changes_count

resourcechanges 
| extend changeTime=todatetime(properties.changeAttributes.timestamp)
| summarize event_count = count() by bin(changeTime, 1m)
| order by event_count
```

<details>
  <summary><b><i>Results</i></b></summary>

![Screen](./img/20220509221312.jpg "Screen")
</details>

#### 3. Ilośc zasobów według lokalizacji
```kql
Resources
| summarize resource_count=count() by tostring(location)
| order by resource_count
```

<details>
  <summary><b><i>Results</i></b></summary>

![Screen](./img/20220509221927.jpg "Screen")
</details>


#### 4. Lista adresów IP używanych przez VM
* [Azure Resource Graph query to find all VMs with public IPs](https://stackoverflow.com/questions/56758465/azure-resource-graph-query-to-find-all-vms-with-public-ips)
* [Print all VM names and private IP from subnet](https://stackoverflow.com/questions/67674782/print-all-vm-names-and-private-ip-from-subnet)
* [List virtual machines with their network interface and public IP](https://docs.microsoft.com/en-us/azure/governance/resource-graph/samples/advanced?tabs=azure-cli#join-vmpip)


```kql
Resources
    | where type =~ 'microsoft.compute/virtualmachines'
    | project vmId = tolower(tostring(id)), vmName = name
    | join (Resources
        | where type =~ 'microsoft.network/networkinterfaces'
        | mv-expand ipconfig=properties.ipConfigurations
        | project vmId = tolower(tostring(properties.virtualMachine.id)), privateIp = ipconfig.properties.privateIPAddress, publicIpId = tostring(ipconfig.properties.publicIPAddress.id)
        | join kind=leftouter (Resources
            | where type =~ 'microsoft.network/publicipaddresses'
            | project publicIpId = id, publicIp = properties.ipAddress
        ) on publicIpId
        | project-away publicIpId, publicIpId1
        | summarize privateIps = make_list(privateIp), publicIps = make_list(publicIp) by vmId
    ) on vmId
    | project-away vmId1
    | sort by vmName asc
| where array_length(publicIps)>0
```

<details>
  <summary><b><i>Results</i></b></summary>

![Screen](./img/20220509223827.jpg "Screen")
</details>


#### 5. Ilość zmian w ciągu dnia
```kql
resourcechanges 
| extend hour = floor(todatetime(properties.changeAttributes.timestamp) % 1d , 1h)
| summarize event_count=count() by hour
| sort by hour asc
//| render timechart
//| render columnchart
```

<details>
  <summary><b><i>Results</i></b></summary>

![Screen](./img/20220510123612.jpg "Screen")
</details>


#### 6. VM private IP
```kql
Resources
| where type =~ 'microsoft.compute/virtualmachines'
| extend nics=array_length(properties.networkProfile.networkInterfaces)
| mv-expand nic=properties.networkProfile.networkInterfaces
| where nics == 1 or nic.properties.primary =~ 'true' or isempty(nic)
| project vmId = id, vmName = name, vmSize=tostring(properties.hardwareProfile.vmSize), nicId = tostring(nic.id)
| join kind=leftouter (
    Resources
    | where type =~ 'microsoft.network/networkinterfaces'
    | extend ipConfigsCount=array_length(properties.ipConfigurations)
    | mv-expand ipconfig=properties.ipConfigurations
    | where ipConfigsCount == 1 or ipconfig.properties.primary =~ 'true'
    | project nicId = id, privateIPAddress = tostring(ipconfig.properties.privateIPAddress))
on nicId
```


<!-- ## Linki
* [Azure REST APIs with Postman (2021) - youtube](https://youtu.be/6b1J03fDnOg)
* [Get the List of All Azure VMs With All Their Private and Public IPs](https://mihai-albert.com/2020/10/01/get-the-list-of-all-azure-vms-with-all-their-private-and-public-ips/)
https://blog.blksthl.com/2022/02/22/list-all-changes-made-in-your-azure-environment-in-one-query-using-azure-resource-graph/

 -->

