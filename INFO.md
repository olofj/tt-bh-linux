# Tenstorrent Blackhole P100/P150 Linux demo

## Additional information

### Device Tree
Sample device tree provided at `x280.dts`. This uses 2896MB of memory for the
host and the remaining 1200MB for the rootfs. The values in the next section
hence assume we're putting the rootfs at 0x4000e5000000 (which is
0x400030000000 + 2896MB)

Compiling device tree `dtc x280.dts > x280.dtb`

### Different methods of booting
1. Running with FW_PAYLOAD, Opensbi has kernel and dtb integrated into it
Opensbi compiled with these args
```
CROSS_COMPILE=riscv64-linux-gnu- make DEBUG=1 FW_PIC=y FW_PAYLOAD_OFFSET=0x200000 PLATFORM=generic FW_PAYLOAD=y FW_PAYLOAD_PATH=<path-to-kernel> FW_FDT_PATH=<path-to-dtb>
```

```
# Adjusted for FS of size 1200 MB
     FS_ADDR="0x4000e5000000"
PAYLOAD_ADDR="0x400030000000"

PAYLOAD=<path-to-opensbi-fw_payload.bin>
FS=<path-to-rootfs-img>

python boot.py --boot --opensbi_bin $PAYLOAD --opensbi_dst $PAYLOAD_ADDR --rootfs_bin $FS --rootfs_dst $FS_ADDR
```

2. Running with FW_JUMP, Opensbi has dtb integrated into it, kernel is separate
Opensbi compiled with these args
```
CROSS_COMPILE=riscv64-linux-gnu- make DEBUG=1 FW_PIC=y FW_JUMP=y FW_JUMP_OFFSET=0x200000 FW_FDT_PATH=<path-to-dtb> PLATFORM=generic
```

```
# Adjusted for FS of size 1200 MB
     FS_ADDR="0x4000e5000000"
PAYLOAD_ADDR="0x400030000000"
 KERNEL_ADDR="0x400030200000"

PAYLOAD=<path-to-opensbi-fw_jump.bin>
FS=<path-to-rootfs-img>
KERNEL=<path-to-kernel-image>

python boot.py --boot --opensbi_bin $PAYLOAD --opensbi_dst $PAYLOAD_ADDR --rootfs_bin $FS --rootfs_dst $FS_ADDR --kernel_bin $KERNEL --kernel_dst $KERNEL_ADDR
```

3. Running with FW_JUMP, Opensbi doesn't have kernel or dtb integrated, both are separate
Note: this doesn't work currently, requires some patching to opensbi to set dtb
location in a1 register before boot (@mikey)

Opensbi compiled with these args
```
CROSS_COMPILE=riscv64-linux-gnu- make DEBUG=1 FW_PIC=y FW_JUMP=y FW_JUMP_OFFSET=0x200000 FW_JUMP_FDT_OFFSET=0x100000  PLATFORM=generic
```

```
# Adjusted for FS of size 1200 MB
     FS_ADDR="0x4000e5000000"
PAYLOAD_ADDR="0x400030000000"
 KERNEL_ADDR="0x400030200000"
    DTB_ADDR="0x400030100000"

PAYLOAD=<path-to-opensbi-fw_jump.bin>
FS=<path-to-rootfs-img>
KERNEL=<path-to-kernel-image>
DTB=<path-to-dtb>

python boot.py --boot --opensbi_bin $PAYLOAD --opensbi_dst $PAYLOAD_ADDR --rootfs_bin $FS --rootfs_dst $FS_ADDR --kernel_bin $KERNEL --kernel_dst $KERNEL_ADDR --dtb_bin $DTB --dtb_dst $DTB_ADDR 

```

4. [TODO] Write an example on putting FS in a remote DRAM tile so that x280
   gets full 4G of memory to itself
