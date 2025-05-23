# Common inventory fields between Linux and Windows

This is tested and works with zabbix_agentd 7.0

## Amount of CPUs

### Linux
Zabbix agent Key:
```
vfs.file.contents[/proc/cpuinfo]
```
JavaScript preprocessing for dependent item
Count amount of processors. JavaScript:
```javascript
return value.match(/^processor/gm).length;
```

### Windows
Zabbix agent Key:
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


## Operating system

### Linux
Zabbix agent Key:
```
vfs.file.contents[/etc/os-release]
```
#### Preprocessing steps for dependent item
Extract only pretty name. Regular expression:
```regex
PRETTY_NAME=.(.*).
```

### Windows
Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_OperatingSystem]
```
#### Preprocessing steps for dependent item
Extract only "Caption". JSONPath:
```jsonpath
$[0].Caption
```


## Disk

### Linux
Zabbix agent Key:
```
vfs.file.contents[/proc/partitions]
```
#### Preprocessing steps for dependent item
Read lines which are not partitions. JavaScript:
```javascript
var input = value.match(/\d+\s+0\s+\d+\s+\S+/gm);
var out = [];
for (var n = 0; n < input.length; n++) {
    var row = {};
    row["disk"] = input[n].match(/\d+\s+0\s+\d+\s+(\S+)/)[1];
    row["size"] = input[n].match(/\d+\s+0\s+(\d+)\s+\S+/)[1];
    out.push(row);
}
return JSON.stringify(out);
```

Ignore "sr0" and "loop0". JSONPath:
```
$..[?(!(@.['disk'] =~ "^sr" || @.['disk'] =~ "^loop"))]
```

Sum total size of all disks together. JSONPath:
```
$[*].size.sum()
```

Convert kilobytes to bytes. Custom multiplier:
```
1024
```


### Windows
Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_DiskDrive]
```

#### Preprocessing steps for dependent item
Ignore model "Microsoft Virtual Disk". JSONPath:
```jsonpath
$..[?(!(@.['Model'] == 'Microsoft Virtual Disk'))]
```

Ignore USB devices. JSONPath:
```jsonpath
$..[?(!(@.['InterfaceType'] == 'USB'))].Size.first()
```








## Total memory

### Linux
Zabbix agent Key:
```
vfs.file.contents[/proc/meminfo]
```
#### Preprocessing steps for dependent item
Read lines which are not partitions. Regular expression:
```regex
MemTotal:\s+(\d+)
```

Convert kilobytes to bytes. Custom multiplier:
```
1024
```


### Windows
Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_PhysicalMemory]
```

#### Preprocessing steps for dependent item
Size of all memory modules in bytes. JSONPath:
```jsonpath
$[*].Capacity.sum()
```
