A lot of this was pulled from https://github.com/bryansteiner/gpu-passthrough-tutorial/#part4 , there are more hooks there. Hugepages is here since I needed to fix it.

### Give your vm some memory and back it with hugepages.
```
<memory unit="KiB">16777216</memory>
<currentMemory unit="KiB">16777216</currentMemory>
<memoryBacking>
    <hugepages/>
</memoryBacking>

```
### QEMU supports hooks so lets get those installed
```
$ sudo wget 'https://raw.githubusercontent.com/PassthroughPOST/VFIO-Tools/master/libvirt_hooks/qemu' \
     -O /etc/libvirt/hooks/qemu
$ sudo chmod +x /etc/libvirt/hooks/qemu
```
### What your hook structure should look like.
```
$ tree /etc/libvirt/hooks/
/etc/libvirt/hooks/
├── kvm.conf
├── qemu
└── qemu.d
    └── win10
        ├── prepare
        │   └── begin
        │       ├── ...
        │       └── alloc_hugepages.sh
        └── release
            └── end
                ├── ...
                └── dealloc_hugepages.sh
```
 ### For standing up hugepages on the fly.
`alloc_hugepages.sh`
```
#!/bin/bash

## Load the config file
source "/etc/libvirt/hooks/kvm.conf"
echo "$MEMORY"

## For the love of god dont use 1gig hugepages, due to memory frag you'll never get the vm rebooted again.
## Calculate number of hugepages to allocate from memory (in MB)
HUGEPAGES=8192
echo "$HUGEPAGES"

echo "Allocating hugepages..."
echo $HUGEPAGES > /proc/sys/vm/nr_hugepages
ALLOC_PAGES=$(cat /proc/sys/vm/nr_hugepages)

TRIES=0
while (( $ALLOC_PAGES != $HUGEPAGES && $TRIES < 1000 ))
do
    sync
    echo 3 > /proc/sys/vm/drop_caches
    echo 1 > /proc/sys/vm/compact_memory
    echo 1 > /proc/sys/vm/compact_memory            ## defrag ram
    echo $HUGEPAGES > /proc/sys/vm/nr_hugepages
    ALLOC_PAGES=$(cat /proc/sys/vm/nr_hugepages)
    echo "Succesfully allocated $ALLOC_PAGES / $HUGEPAGES"
    let TRIES+=1
done

if [ "$ALLOC_PAGES" -ne "$HUGEPAGES" ]
then
    echo "Not able to allocate all hugepages. Reverting..."
    echo 0 > /proc/sys/vm/nr_hugepages
    exit 1
fi                                                                                                                                            
```
`dealloc_hugepages.sh`
```
#!/bin/bash

## Load the config file
source "/etc/libvirt/hooks/kvm.conf"

echo 0 > /proc/sys/vm/nr_hugepages
```


