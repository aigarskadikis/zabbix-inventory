<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Linux/Windows common inventory fields](#linuxwindows-common-inventory-fields)
  - [Amount of CPUs](#amount-of-cpus)
    - [Linux](#linux)
      - [Preprocessing steps](#preprocessing-steps)
    - [Windows](#windows)
      - [Preprocessing steps](#preprocessing-steps-1)
  - [Operating system](#operating-system)
    - [Linux](#linux-1)
      - [Preprocessing steps](#preprocessing-steps-2)
    - [Windows](#windows-1)
      - [Preprocessing steps](#preprocessing-steps-3)
  - [Disk](#disk)
    - [Linux](#linux-2)
      - [Preprocessing steps](#preprocessing-steps-4)
    - [Windows](#windows-2)
      - [Preprocessing steps](#preprocessing-steps-5)
  - [Total memory](#total-memory)
    - [Linux](#linux-3)
      - [Preprocessing steps for dependent item](#preprocessing-steps-for-dependent-item)
    - [Windows](#windows-3)
      - [Preprocessing steps for dependent item](#preprocessing-steps-for-dependent-item-1)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

# Linux/Windows common inventory fields

This is tested and works with zabbix_agentd 7.0

## Amount of CPUs

### Linux
Native Zabbix agent Key:
```
vfs.file.contents[/proc/cpuinfo]
```
#### Preprocessing steps
Count of processors. JavaScript:
```javascript
return value.match(/^processor/gm).length;
```

### Windows
Native Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_Processor]
```
#### Preprocessing steps
If "ThreadCount" then use that as amount of processors, otherwise use "NumberOfLogicalProcessors". JavaScript:
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
Native Zabbix agent Key:
```
vfs.file.contents[/etc/os-release]
```
#### Preprocessing steps
Extract only pretty name. Regular expression:
```regex
PRETTY_NAME=.(.*).
```

### Windows
Native Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_OperatingSystem]
```
#### Preprocessing steps
Extract only "Caption". JSONPath:
```jsonpath
$[0].Caption
```


## Disk

### Linux
Native Zabbix agent Key:
```
vfs.file.contents[/proc/partitions]
```
#### Preprocessing steps
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
Native Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_DiskDrive]
```

#### Preprocessing steps
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
