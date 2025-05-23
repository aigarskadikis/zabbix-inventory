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
vfs.file.contents[/proc/meminfo,]
```
#### Preprocessing steps for dependent item
Read lines which are not partitions. Regular expression:
```regex
MemTotal:\s+([0-9]+)
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



## Swap/page file

### Linux
Zabbix agent Key:
```
vfs.file.contents[/proc/meminfo,]
```
#### Preprocessing steps for dependent item
Read lines which are not partitions. Regular expression:
```regex
SwapTotal:\s+(\d+)
```
Convert kilobytes to bytes. Custom multiplier:
```
1024
```


### Windows
Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_OperatingSystem]
```
#### Preprocessing steps for dependent item
Size of paging file. JSONPath:
```jsonpath
$[0].SizeStoredInPagingFiles
```
Convert kilobytes to bytes. Custom multiplier:
```
1024
```


## Version of Zabbix agent

### Linux/Windows
Zabbix agent Key:
```
agent.version
```
#### Preprocessing steps for dependent item
Read lines which are not partitions. Regular expression:
```javascript
// allow version to be sorted alphabetically
var major = value.replace(/\.[0-9]+$/,'');
var minor = value.match(/([0-9]+)$/)[1];
if (minor > 9) { return major + '.' + minor } else { return major + '.0' + minor }
```







## IP address

### Linux
Zabbix agent Key:
```
vfs.file.contents[/proc/net/fib_trie,]
```
#### Preprocessing steps for dependent item
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


### Windows
Zabbix agent Key:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_NetworkAdapterConfiguration]
```
#### Preprocessing steps for dependent item
Extract IP address. JSONPath:
```jsonpath
$..[?(@.['IPAddress'])].IPAddress[0].first()
```


