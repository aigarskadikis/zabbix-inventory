<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
# Audit Linux/Windows servers/workstations

- [Dashboard](#create-dashboard)
- [Operating system](#operating-system)
- [CPU architecture](#cpu-architecture)
- [Amount of CPUs](#cpus)
- [CPU model](#cpu-model)
- [Total memory](#total-memory)
- [Swap/page file](#swappage-file)
- [Disk](#disk)
- [Boot time](#boot-time)
- [Online](#online)
- [Version of Zabbix agent](#version-of-zabbix-agent)
- [IP address](#ip-address)



<!-- END doctoc generated TOC please keep comment here to allow auto update -->


This is tested and works with zabbix_agentd 7.0 and zabbix_agent2 7.0

## Create dashboard

Operating system
```
{INVENTORY.OS.SHORT}
```
Arch
```
{INVENTORY.HW.ARCH}
```
CPUs
```
{INVENTORY.SERIALNO.A}
```



## Operating system

**Linux**

Native Zabbix agent Key:
```mathematica
vfs.file.contents["/etc/os-release",]
```
**Preprocessing steps for dependent item**

Populates host inventory field "OS (short)"

Extract only pretty name. Regular expression:
```
PRETTY_NAME=.(.*).
```

**Windows**

Native Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_OperatingSystem"]
```
**Preprocessing steps for dependent item**

Populates host inventory field "OS (short)"

Extract only "Caption". JSONPath:
```javascript
$[0].Caption
```


## CPU architecture

**Linux/Windows**

Native Zabbix agent Key:
```mathematica
system.uname
```
**Preprocessing steps for dependent item**

Populates host inventory field "HW architecture"


JavaScript:
```javascript
var patterns = [
    { regex: /x86_64/gm, result: 'x64_86' },
    { regex: /aarch64/gm, result: 'aarch64' },
    { regex: /x86$/gm, result: 'x86' },
    { regex: / x64/gm, result: 'x64_86' }
];

for (var i = 0; i < patterns.length; i++) {
    if (value.match(patterns[i].regex)) {
        return patterns[i].result;
    }
}

return value;
```



## CPUs

**Linux**

Native Zabbix agent Key:
```mathematica
vfs.file.contents["/proc/cpuinfo",]
```
**Preprocessing steps for dependent item**

Populates host inventory field "Serial number A"


Count of processors. JavaScript:
```javascript
return value.match(/^processor/gm).length;
```

**Windows**

Native Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_Processor"]
```
**Preprocessing steps for dependent item**

Populates host inventory field "Serial number A"


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




## CPU model

**Linux**

Native Zabbix agent Key:
```mathematica
vfs.file.contents["/proc/cpuinfo",]
```
**Preprocessing steps for dependent item**

On x64 Linux extract "CPU name", on aarch64 extract "CPU part". JavaScript:
```javascript
// extract "model name" or "CPU part" code

// in case master item is stored in inventory, need to remove tabs and new line characters
var formated = value.replace(/\\t/gm, '').replace(/\\n/gm, '\n');

// for x86_64
if (formated.match(/model nam.*: (.*)/)) { return formated.match(/model nam.*: (.*)/)[1]; }

// for aarch64
if (formated.match(/CPU par.*: (.*)/)) { return formated.match(/CPU par.*: (.*)/)[1]; }
```

**Windows**

Native Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_OperatingSystem"]
```
**Preprocessing steps for dependent item**

Extract only "Caption". JSONPath:
```javascript
$[0].Caption
```



## Total memory

**Linux**

Zabbix agent Key:
```mathematica
vfs.file.contents["/proc/meminfo",]
```
**Preprocessing steps for dependent item**

Read lines which are not partitions. Regular expression:
```
MemTotal:\s+([0-9]+)
```

Convert kilobytes to bytes. Custom multiplier:
```
1024
```


**Windows**

Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_PhysicalMemory"]
```

**Preprocessing steps for dependent item**

Size of all memory modules in bytes. JSONPath:
```javascript
$[*].Capacity.sum()
```



## Swap/page file

**Linux**

Zabbix agent Key:
```mathematica
vfs.file.contents["/proc/meminfo",]
```
**Preprocessing steps for dependent item**

Read lines which are not partitions. Regular expression:
```
SwapTotal:\s+(\d+)
```
Convert kilobytes to bytes. Custom multiplier:
```
1024
```


**Windows**

Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_OperatingSystem"]
```
**Preprocessing steps for dependent item**

Size of paging file. JSONPath:
```javascript
$[0].SizeStoredInPagingFiles
```
Convert kilobytes to bytes. Custom multiplier:
```
1024
```







## Disk

**Linux**

Native Zabbix agent Key:
```mathematica
vfs.file.contents["/proc/partitions",]
```
**Preprocessing steps for dependent item**

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
```javascript
$..[?(!(@.['disk'] =~ "^sr" || @.['disk'] =~ "^loop"))]
```

Sum total size of all disks together. JSONPath:
```javascript
$[*].size.sum()
```

Convert kilobytes to bytes. Custom multiplier:
```
1024
```


**Windows**

Native Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_DiskDrive"]
```

**Preprocessing steps for dependent item**

Ignore model "Microsoft Virtual Disk". JSONPath:
```javascript
$..[?(!(@.['Model'] == 'Microsoft Virtual Disk'))]
```

Ignore USB devices. JSONPath:
```javascript
$..[?(!(@.['InterfaceType'] == 'USB'))].Size.first()
```


## Boot time

**Linux**

Native Zabbix agent Key:
```mathematica
system.boottime
```

**Preprocessing steps for dependent item**

```javascript
// convert unixtime to human readable
function pad(n) { return n < 10 ? '0' + n : n }

function formatDate(unix) {
    var d = new Date(unix * 1000);
    return d.getFullYear()
        + '.' + pad(d.getMonth() + 1)
        + '.' + pad(d.getDate())
        + ' ' + pad(d.getHours())
        + ':' + pad(d.getMinutes())
        + ':' + pad(d.getSeconds())
}

return formatDate(value);
```


**Windows**

Native Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_OperatingSystem"]
```
**Preprocessing steps for dependent item**

JSONPath:
```javascript
$[0].LastBootUpTime
```
Regular expression:
```
^(....)(..)(..)
```
Extract \1.\2.\3



## Online

**Linux/Windows**

Native Zabbix agent Key:
```mathematica
system.uptime
```

Item type "Calculated":
```mathematica
count(//system.uptime,91s)
```

**Preprocessing steps for dependent item**

JavaScript:
```javascript
if (value > 0) {return 1} else {return 0}
```
Discard unchanged with heartbeat: 1d




## Version of Zabbix agent

**Linux/Windows**

Native Zabbix agent Key:
```mathematica
agent.version
```
**Preprocessing steps for dependent item**

Read lines which are not partitions. Regular expression:
```javascript
// allow version to be sorted alphabetically
var major = value.replace(/\.[0-9]+$/,'');
var minor = value.match(/([0-9]+)$/)[1];
if (minor > 9) { return major + '.' + minor } else { return major + '.0' + minor }
```




## IP address

**Linux**

Zabbix agent Key:
```mathematica
vfs.file.contents["/proc/net/fib_trie",]
```
**Preprocessing steps for dependent item**

Ignore IPs which start with "127" or "169". Ignore IPs which ends with "0", "1", "254", "255". JavaScript:
```javascript
// remove new line characters, leave only printable characters
var format = value.replace(/\\n/gm, "").replace(/[\x00-\x1F\x7F]/g, "");

// extract ip
var data = format.match(/(\S+)\s+\/32/gm);

if (data.length > 1) {
    data = format.match(/(\S+)\s+\/32/gm).sort();
}
function uniq(a) {
    return a.sort().filter(function (item, pos, ary) {
        return !pos || item != ary[pos - 1];
    });
}
var out = [];
for (n = 0; n < data.length; n++) {
    if (!
        (data[n].match(/^127/))
        && !(data[n].match(/\S+\.255 /))
        && !(data[n].match(/\S+\.254 /))
        && !(data[n].match(/\S+\.0 /))
        && !(data[n].match(/\S+\.1 /))
        && !(data[n].match(/^169/))
    ) {
        out.push(data[n].match(/^\S+/)[0]);
    }
}
var filter = uniq(out);
return filter[0];
```


**Windows**

Zabbix agent Key:
```mathematica
wmi.getall["root\cimv2", "SELECT * FROM Win32_NetworkAdapterConfiguration"]
```
**Preprocessing steps for dependent item**

Extract IP address. JSONPath:
```javascript
$..[?(@.['IPAddress'])].IPAddress[0].first()
```






