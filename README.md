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

Populates host inventory field "Contract number", later access with:
```
{INVENTORY.CONTRACT.NUMBER}
```

## Operating system

### Linux
```
vfs.file.contents[/etc/os-release]
```
Preprocessing steps for dependent item
Regular expression
```regex
PRETTY_NAME=.(.*).
```

### Windows
```
wmi.getall[root\cimv2,SELECT * FROM Win32_OperatingSystem]
```
Preprocessing steps for dependent item
JSONPath
```jsonpath
$[0].Caption
```
Replace "Microsoft " with nothing

Populates host inventory field "Type", later access with:
```
{INVENTORY.TYPE}
```


## Architecture

### Linux
```
system.uname
```
JavaScript preprocessing for dependent item
```javascript

```

### Windows
```
system.uname
```
Json path preprocessing for dependent item:
```javascript

```


Populates host inventory field "HW architecture", later access with:
```
{INVENTORY.HW.ARCH}
```


