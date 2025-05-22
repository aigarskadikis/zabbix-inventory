# Collection to obtain data from Windows/Linux servers/workstations

This is tested and works with Zabbix 7.0 agentd(agent1)

## Amount of CPUs

Item key on Linux:
```
vfs.file.contents[/proc/cpuinfo]
```
JavaScript preprocessing for dependent item
```javascript
return value.match(/^processor/gm).length;
```

Item key on Windows:
```
wmi.getall[root\cimv2,SELECT * FROM Win32_Processor]
```
JavaScript preprocessing for dependent item
```javascript
// locate ThreadCount, but if it not exists, report NumberOfLogicalProcessors
var input = JSON.parse(value)[0];
if (input.ThreadCount) {return input.ThreadCount}
else {return input.NumberOfLogicalProcessors}
```

