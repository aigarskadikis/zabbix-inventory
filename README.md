# Collection to obtain data from Windows/Linux servers/workstations

This is tested and works with Zabbix 7.0 agentd(agent1)

## Amount of CPUs

### Linux
```
vfs.file.contents[/proc/cpuinfo]
```
JavaScript preprocessing for dependent item
```javascript
return value.match(/^processor/gm).length;
```

### Windows
```
wmi.getall[root\cimv2,SELECT * FROM Win32_Processor]
```
JavaScript preprocessing for dependent item
```javascript
// locate ThreadCount, but if it not exists, report NumberOfLogicalProcessors
var input = JSON.parse(value)[0];
if (input.ThreadCount) {
return input.ThreadCount;
} else {
return input.NumberOfLogicalProcessors;
}
```

Store in "Contract number", later access with:
```
{INVENTORY.CONTRACT.NUMBER}
```

## Operating system

### Linux
```
vfs.file.contents[/etc/os-release]
```
JavaScript preprocessing for dependent item
```regex
PRETTY_NAME=.(.*).
```

### Windows
```
wmi.getall[root\cimv2,SELECT * FROM Win32_OperatingSystem]
```
Json path preprocessing for dependent item:
```jsonpath
$[0].Caption
```

Store in "Type", later access with:
```
{INVENTORY.TYPE}
```

