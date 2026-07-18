---
title: "自作 OS で memcmp"
date: 2023-10-12T17:57:25+09:00
draft: false
---

## C 標準ライブラリ実装

教材には memcpy や strcmp が実装されていたので，練習がてら memcmp を実装する

```C
int memcmp(const void *buf1, const void *buf2, size_t n){
    const uint8_t *p = (const uint8_t *)buf1;
    const uint8_t *t = (const uint8_t *)buf2;
    while(n--){
      if(*p != *t)
        return *p - *t;
      p++;
      t++;
    }
    return 0;
}
```

## テストの追加

そういえばテストが無いので追加する．

```C
void test_libc(void) {
    assert(memcmp("abc", "abc", 3) == 0);
    assert(memcmp("abc", "abd", 3) < 0);
    
    // strcmp
    assert(strcmp("hello", "hello") == 0);
    assert(strcmp("abc", "abd") < 0);
    assert(strcmp("apple", "app") > 0);
    
    // memcpy
    char buf[10];
    memcpy(buf, "hello", 6);
    assert(strcmp(buf, "hello") == 0);
    
    
    printf("All tests passed\n");
}
```

あとは assert

```C
#define assert(expr) \
    do { \
        if (!(expr)) { \
            printf("Assertion failed: %s at %s:%d\n", #expr, __FILE__, __LINE__); \
            for (;;); \
        } \
    } while (0)
```

実行結果
```sh
u@u-VirtualBox:~/French_Cruller$ ./run.sh 
+ QEMU=qemu-system-riscv32
+ CC=clang
+ CFLAGS='-std=c11 -O2 -g3 -Wall -Wextra --target=riscv32-unknown-elf -fuse-ld=lld -fno-stack-protector -ffreestanding -nostdlib'
+ clang -std=c11 -O2 -g3 -Wall -Wextra --target=riscv32-unknown-elf -fuse-ld=lld -fno-stack-protector -ffreestanding -nostdlib -Wl,-Tkernel.ld -Wl,-Map=kernel.map -o kernel.elf kernel.c common.c
+ qemu-system-riscv32 -machine virt -bios default -nographic -serial mon:stdio --no-reboot -kernel kernel.elf

OpenSBI v1.7
   ____                    _____ ____ _____
  / __ \                  / ____|  _ \_   _|
 | |  | |_ __   ___ _ __ | (___ | |_) || |
 | |  | | '_ \ / _ \ '_ \ \___ \|  _ < | |
 | |__| | |_) |  __/ | | |____) | |_) || |_
  \____/| .__/ \___|_| |_|_____/|____/_____|
        | |
        |_|

Platform Name               : riscv-virtio,qemu
Platform Features           : medeleg
Platform HART Count         : 1
Platform IPI Device         : aclint-mswi
Platform Timer Device       : aclint-mtimer @ 10000000Hz
Platform Console Device     : uart8250
Platform HSM Device         : ---
Platform PMU Device         : ---
Platform Reboot Device      : syscon-reboot
Platform Shutdown Device    : syscon-poweroff
Platform Suspend Device     : ---
Platform CPPC Device        : ---
Firmware Base               : 0x80000000
Firmware Size               : 313 KB
Firmware RW Offset          : 0x40000
Firmware RW Size            : 57 KB
Firmware Heap Offset        : 0x45000
Firmware Heap Size          : 37 KB (total), 2 KB (reserved), 10 KB (used), 24 KB (free)
Firmware Scratch Size       : 4096 B (total), 1340 B (used), 2756 B (free)
Runtime SBI Version         : 3.0
Standard SBI Extensions     : time,rfnc,ipi,base,hsm,srst,pmu,dbcn,fwft,legacy,dbtr,sse
Experimental SBI Extensions : none

Domain0 Name                : root
Domain0 Boot HART           : 0
Domain0 HARTs               : 0*
Domain0 Region00            : 0x00100000-0x00100fff M: (I,R,W) S/U: (R,W)
Domain0 Region01            : 0x10000000-0x10000fff M: (I,R,W) S/U: (R,W)
Domain0 Region02            : 0x02000000-0x0200ffff M: (I,R,W) S/U: ()
Domain0 Region03            : 0x80040000-0x8004ffff M: (R,W) S/U: ()
Domain0 Region04            : 0x80000000-0x8003ffff M: (R,X) S/U: ()
Domain0 Region05            : 0x0c400000-0x0c5fffff M: (I,R,W) S/U: (R,W)
Domain0 Region06            : 0x0c000000-0x0c3fffff M: (I,R,W) S/U: (R,W)
Domain0 Region07            : 0x00000000-0xffffffff M: () S/U: (R,W,X)
Domain0 Next Address        : 0x80200000
Domain0 Next Arg1           : 0x87e00000
Domain0 Next Mode           : S-mode
Domain0 SysReset            : yes
Domain0 SysSuspend          : yes

Boot HART ID                : 0
Boot HART Domain            : root
Boot HART Priv Version      : v1.12
Boot HART Base ISA          : rv32imafdch
Boot HART ISA Extensions    : sstc,zicntr,zihpm,zicboz,zicbom,sdtrig,svadu
Boot HART PMP Count         : 16
Boot HART PMP Granularity   : 2 bits
Boot HART PMP Address Bits  : 32
Boot HART MHPM Info         : 16 (0x0007fff8)
Boot HART Debug Triggers    : 2 triggers
Boot HART MIDELEG           : 0x00001666
Boot HART MEDELEG           : 0x00f0b509
All tests passed
```

OK
