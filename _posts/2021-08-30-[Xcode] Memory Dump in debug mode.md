---
title: "[Xcode] Memory Dump in debug mode"
excerpt: "Memory Dump"
description: "Memory Dump"
modified: 2021-08-30
categories: "iOS"
tags: [Xcode, Memory, dump, lldb]
---

# Memory dump
## Create a python script

```python
import lldb
def processAllMemoryRegions():
    process = lldb.debugger.GetSelectedTarget().GetProcess()
    memoryRegionInfoList = process.GetMemoryRegions()
    numberOfMemoryRegions = memoryRegionInfoList.GetSize()
    memoryRegionIndex = 0
    while (memoryRegionIndex < numberOfMemoryRegions):
        memoryRegionInfo = lldb.SBMemoryRegionInfo()
        success = memoryRegionInfoList.GetMemoryRegionAtIndex(memoryRegionIndex, memoryRegionInfo)
        if success:
            print("Processing: "+str(memoryRegionIndex+1)+"/"+str(numberOfMemoryRegions))
            processOneMemoryRegion(process, memoryRegionInfo)
        else:
            print("Could not get memory at index: "+str(memoryRegionIndex))
        memoryRegionIndex = memoryRegionIndex+1
def processOneMemoryRegion(process, memoryRegionInfo):
    begAddressOfMemoryRegion = memoryRegionInfo.GetRegionBase()
    endAddressOfMemoryRegion = memoryRegionInfo.GetRegionEnd()
    if memoryRegionInfo.IsReadable():
        print("Beg address of a memory region: "+stringifyMemoryAddress(begAddressOfMemoryRegion))
        print("End address of a memory region: "+stringifyMemoryAddress(endAddressOfMemoryRegion))
        error = lldb.SBError()
        regionSize = endAddressOfMemoryRegion-begAddressOfMemoryRegion
        memoryData = process.ReadMemory(begAddressOfMemoryRegion, regionSize, error)
        if error.Success():
            #do something with memoryData (bytearray) eg. save it to file
            file = open("/absolute path/sample.bin", "ab")
            file.write(memoryData)
            file.close()
            pass
        else:
            print("Could not access memory data.")
    else:
        print("Memory region is not readable.")
def stringifyMemoryAddress(memoryAddress):
    return '0x{:016x}'.format(memoryAddress)
```

## lldb

```
(lldb) script
Python Interactive Interpreter. To exit, type 'quit()', 'exit()'.
>>> exec (open ( '/absolute path/mem.py'). read ())
>>> processAllMemoryRegions ()
```

# Use hexdump Command
```shell
$ hexdump -C sample.bin

00000c00  48 c7 04 32 00 00 00 00  31 c0 83 c9 ff f0 0f b1  |H..2....1.......|
00000c10  4f 10 74 1d f3 90 31 c0  f0 0f b1 4f 10 74 12 f3  |O.t...1....O.t..|
```

# Reference
- [https://www.python2.net/questions-668418.htm](https://www.python2.net/questions-668418.htm){:target="_blank"}
- [https://linuxhint.com/how-to-use-hexdump-command-in-linux/](https://linuxhint.com/how-to-use-hexdump-command-in-linux/){:target="_blank"}
