# Exploring the Security Features of MetaComputing AI PC with Framework Laptop 13: MTE (Memory Tagging Extension)
MTE is a hardware-level memory security technology introduced with ARMv9 architecture. It assigns a 4-bit random tag to each 16-byte memory block (with a matching tag embedded in the pointer synchronously), and the hardware automatically verifies tag consistency during each memory access. 

Its core value lies in efficiently intercepting critical vulnerabilities such as buffer overflows, use-after-free (UAF), and wild pointer dereferences (covering over 90% of memory attacks in actual tests), and implements crash prevention or asynchronous auditing with near-zero performance loss (<2%). Currently, it has been deployed on a large scale in the Android system core layer, Linux kernel, and AWS cloud servers, driving memory security from passive vulnerability patching to active attack prevention.

For this experiment, we will conduct in-depth testing based on the upcoming MetaComputing AI PC with Framework Laptop 13 device from MetaComputing. This device is equipped with the CP8180 Arm architecture processor, as a core product filling the Arm architecture gap in the Framework modular hardware ecosystem. It features a CPU design with 8 Arm Cortex-A720 cores + 4 Arm Cortex-A520 cores, fits the standard Framework Laptop 13 body, and comes pre-installed with Ubuntu 25.04 aarch64 system. 

Its native excellent support for Arm architecture hardware security features provides an ideal hardware carrier and highly compatible software environment for us to explore the actual protection effectiveness of MTE technology.

## System Environment
- MetaComputing AI PC: https://metacomputing.io/collections/frontpage/products/metacomputing-aipc
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/54e994ca-1844-4891-a5c4-82c055010ffd" />


## Enabling the MTE Function
- The MTE function is disabled by default and needs to be manually enabled.
- Steps to enable the MTE function:
1. Press the Esc key during device boot-up to enter the BIOS environment, as shown in the figure below 
<img width="1280" height="960" alt="image" src="https://github.com/user-attachments/assets/79f1a90c-1721-42f4-9ff0-e0df1e4a2ff8" />

2. Select the MetaComputing System Manager option to enter, as shown in the figure below.
<img width="1280" height="770" alt="image" src="https://github.com/user-attachments/assets/3e7e429f-2c43-47e9-8f57-7acc9fcadf61" />

3. Select SE Configuration to enter the MTE switch settings, and set the switch to Enabled. 
<img width="1280" height="790" alt="image" src="https://github.com/user-attachments/assets/d25d2d0d-866e-4d66-94d0-9ef08b6cc5d8" />

4. Press F10 to save after setting, and restart the device for the settings to take effect

## Exploring the MTE Function
### Test Objectives
- Enable MTE synchronous detection mode
- Allocate memory pages with tag protection
- Actively tag pointers and write memory tags
- Perform legal access (should succeed)
- Construct illegal access with tag mismatch (should trigger SIGSEGV)
- Capture exceptions and determine them as MTE synchronization errors (SEGV_MTESERR)

MTE Function Testing
- MTE Function Check

```
zcat /proc/config.gz | grep CONFIG_ARM64_MTE
CONFIG_ARM64_MTE=y
```

- Check GCC compiler version: gcc -v. If prompted "command not found", install it via the package manager first (e.g., `sudo apt install gcc`` on Ubuntu).
```
$roma@roma-MC-FML13V04-Board ~> gcc -v
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/gcc/aarch64-linux-gnu/14/lto-wrapper
OFFLOAD_TARGET_NAMES=nvptx-none
OFFLOAD_TARGET_DEFAULT=1
Target: aarch64-linux-gnu
Configured with: ../src/configure -v --with-pkgversion='Ubuntu 14.2.0-19ubuntu2' --with-bugurl=file:///usr/share/doc/gcc-14/README.Bugs --enable-languages=c,ada,c++,go,d,fortran,objc,obj-c++,m2,rust --prefix=/usr --with-gcc-major-version-only --program-suffix=-14 --program-prefix=aarch64-linux-gnu- --enable-shared --enable-linker-build-id --libexecdir=/usr/libexec --without-included-gettext --enable-threads=posix --libdir=/usr/lib --enable-nls --enable-bootstrap --enable-clocale=gnu --enable-libstdcxx-debug --enable-libstdcxx-time=yes --with-default-libstdcxx-abi=new --enable-libstdcxx-backtrace --enable-gnu-unique-object --disable-libquadmath --disable-libquadmath-support --enable-plugin --enable-default-pie --with-system-zlib --enable-libphobos-checking=release --with-target-system-zlib=auto --enable-objc-gc=auto --enable-multiarch --enable-fix-cortex-a53-843419 --disable-werror --enable-offload-targets=nvptx-none=/build/gcc-14-g1I132/gcc-14-14.2.0/debian/tmp-nvptx/usr --enable-offload-defaulted --without-cuda-driver --enable-checking=release --build=aarch64-linux-gnu --host=aarch64-linux-gnu --target=aarch64-linux-gnu --with-build-config=bootstrap-lto-lean --enable-link-serialization=4
Thread model: posix
Supported LTO compression algorithms: zlib zstd
gcc version 14.2.0 (Ubuntu 14.2.0-19ubuntu2)
```

- Prepare Test Code
```
#include <stdio.h>
#include <stdlib.h>
#include <sys/mman.h>
#include <sys/prctl.h>
#include <unistd.h>
#include <signal.h>
#include<string.h>

/* MTE-related macros (fallback definitions if not in system headers) */
#ifndef PR_SET_TAGGED_ADDR_CTRL
#define PR_SET_TAGGED_ADDR_CTRL 55
#define PR_GET_TAGGED_ADDR_CTRL 56
#define PR_TAGGED_ADDR_ENABLE (1UL << 0)
#define PR_MTE_TCF_SHIFT 1
#define PR_MTE_TCF_SYNC (1UL << PR_MTE_TCF_SHIFT)
#define PR_MTE_TCF_ASYNC (2UL << PR_MTE_TCF_SHIFT)
#define PR_MTE_TAG_SHIFT 3
#endif

#ifndef PROT_MTE
#define PROT_MTE 0x20
#endif

/* Signal handler for MTE faults */
void mte_sig_handler(int sig, siginfo_t *si, void *unused) {
    printf("\n[MTE TEST] ---> SIGSEGV Received!\n");
    printf("|_ Fault Address: %p\n", si->si_addr);
    printf("|_ Error Code: 0x%x\n", si->si_code);

    // MTE-specific error codes (Linux 5.10+)
    #ifdef SEGV_MTESERR
    if (si->si_code == SEGV_MTESERR) {
        printf("|_ MTE Sync Tag Check Failed!\n");
        printf("[TEST RESULT] MTE PROTECTION ACTIVE - TEST PASSED\n");
        exit(0);
    }
    #endif

    printf("[TEST RESULT] UNKNOWN ERROR - TEST FAILED\n");
    exit(1);
}

int main() {
    printf("\n===== ARM64 MTE FUNCTIONAL TEST =====\n");

    // Register signal handler
    struct sigaction sa;
    sa.sa_flags = SA_SIGINFO;
    sa.sa_sigaction = mte_sig_handler;
    sigemptyset(&sa.sa_mask);
    sigaction(SIGSEGV, &sa, NULL);

    // Verify hardware support
    printf("\n[PHASE 1] Hardware/Kernel Support Check\n");
    FILE *cpuinfo = fopen("/proc/cpuinfo", "r");
    if (cpuinfo) {
        char line[256];
        int mte_found = 0;
        while (fgets(line, sizeof(line), cpuinfo)) {
            if (strstr(line, "mte") || strstr(line, "mte3")) {
                printf("|_ CPU Feature: %s", line);
                mte_found = 1;
            }
        }
        fclose(cpuinfo);
        if (!mte_found) {
            printf("[ERROR] MTE not supported in CPU features\n");
            return 1;
        }
    }
    printf("|_ Status: Hardware support verified\n");

    // Enable MTE
    printf("\n[PHASE 2] Enable MTE via prctl()\n");
    unsigned long mte_ctrl = PR_TAGGED_ADDR_ENABLE |
                            PR_MTE_TCF_SYNC |
                            (0xfffe << PR_MTE_TAG_SHIFT);

    if (prctl(PR_SET_TAGGED_ADDR_CTRL, mte_ctrl, 0, 0, 0)) {
        perror("|_ prctl(PR_SET_TAGGED_ADDR_CTRL) failed");
        printf("[ERROR] Kernel may lack CONFIG_ARM64_MTE support\n");
        return 1;
    }
    printf("|_ Status: MTE enabled (sync mode)\n");

    // Allocate MTE-protected memory
    printf("\n[PHASE 3] Memory Allocation with PROT_MTE\n");
    size_t page_size = getpagesize();
    void *ptr = mmap(NULL, page_size,
                    PROT_READ | PROT_WRITE | PROT_MTE,
                    MAP_PRIVATE | MAP_ANONYMOUS,
                    -1, 0);

    if (ptr == MAP_FAILED) {
        perror("|_ mmap() failed");
        return 1;
    }
    printf("|_ Allocated Address: %p\n", ptr);

    // Generate and apply memory tag
    printf("\n[PHASE 4] Apply Memory Tag\n");
    asm volatile(
        "irg %0, %0\n"   // Generate random tag
        : "+r"(ptr)
    );
    asm volatile(
        "stg %0, [%0]\n" // Store tag to memory
        :
        : "r"(ptr)
    );
    printf("|_ Tagged Pointer: %p\n", ptr);
    printf("|_ Tag Value: 0x%lx\n", (unsigned long)ptr >> 56);

    // Valid access test
    printf("\n[PHASE 5] Valid Access Test\n");
    *(int *)ptr = 0xDEADBEEF;
    printf("|_ Write Value: 0x%x\n", 0xDEADBEEF);
    printf("|_ Read Value: 0x%x\n", *(int *)ptr);
    printf("|_ Status: Valid access successful\n");

    // Invalid access test
    printf("\n[PHASE 6] Invalid Access Test\n");
    void *bad_ptr = (void *)((unsigned long)ptr ^ (1UL << 56)); // Flip tag bit
    printf("|_ Corrupted Pointer: %p\n", bad_ptr);
    printf("|_ Attempting access...\n");
    fflush(stdout);

    // This should trigger SIGSEGV
    *(int *)bad_ptr = 0xCAFEBABE;

    // Should never reach here if MTE works
    printf("[TEST RESULT] MTE FAILED TO TRIGGER!\n");
    return 1;
}
```

- Compilation and Testing
```
roma@roma-MC-FML13V04-Board ~> gcc -march=armv8.5-a+memtag MTE_TEST.c -o mte_test 
roma@roma-MC-FML13V04-Board ~> ./mte_test 
 
===== ARM64 MTE FUNCTIONAL TEST ===== 
 
[PHASE 1] Hardware/Kernel Support Check 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ CPU Feature: t svei8mm svebf16 i8mm bf16 dgh bti mte ecv afp mte3 wfxt 
|_ Status: Hardware support verified 
 
[PHASE 2] Enable MTE via prctl() 
|_ Status: MTE enabled (sync mode) 
 
[PHASE 3] Memory Allocation with PROT_MTE 
|_ Allocated Address: 0xffffa0f80000 
 
[PHASE 4] Apply Memory Tag 
|_ Tagged Pointer: 0x900ffffa0f80000 
|_ Tag Value: 0x9 
 
[PHASE 5] Valid Access Test 
|_ Write Value: 0xdeadbeef 
|_ Read Value: 0xdeadbeef 
|_ Status: Valid access successful 
 
[PHASE 6] Invalid Access Test 
|_ Corrupted Pointer: 0x800ffffa0f80000 
|_ Attempting access... 
 
[MTE TEST] ---> SIGSEGV Received! 
|_ Fault Address: 0xffffa0f80000 
|_ Error Code: 0x9 
|_ MTE Sync Tag Check Failed! 
[TEST RESULT] MTE PROTECTION ACTIVE - TEST PASSED 
```
The test successfully verified the hardware (CPU supports MTE features) and kernel functions. After enabling MTE synchronous mode via prctl(), we allocated memory with the PROT_MTE flag (address 0xffffa0f80000) and applied a tag (0x9). In the valid access test, read and write operations worked normally (the value 0xdeadbeef matched), while in the invalid access test (using the corrupted pointer 0x800ffffa0f80000), the system correctly triggered the SIGSEGV signal (error code 0x9), indicating that the MTE synchronous tag checking mechanism effectively intercepted illegal memory operations. The test finally passed (MTE PROTECTION ACTIVE), proving that the memory security protection function of MTE has been activated and runs reliably.

### Conclusion: Making Security the Default Attribute of Computing
- This experiment fully demonstrates that on devices supporting the ARMv9 architecture, MTE can achieve efficient and reliable memory access control at an extremely low cost. Its successful operation means we are shifting from the traditional "passive vulnerability patching" model to a new paradigm of "active attack prevention".
- Just as encryption has become the standard for network communication, memory security should also become a basic capability of all computing platforms. The emergence of MTE is the technical embodiment of this vision. As the ARM ecosystem continues to expand, especially with the rise of emerging fields such as AI PCs, edge computing, and autonomous driving, MTE will no longer be an "optional feature", but will become one of the core indicators for measuring system trustworthiness.

## Expansion
### Potential Impacts After Enabling the MTE Function
While the MTE function brings benefits such as memory security protection to the system, it will occupy part of the system memory resources. After enabling this function, the system will reserve a specific area of memory to support its operation. This approximately 2Gi of memory will not be available for general access for the time being. Please pay attention to this memory allocation change during use and plan the use of system resources rationally~

- Available Memory Before Enabling the Function 
<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/80ae683a-4772-410f-9710-83201377a925" />

- Available Memory After Enabling the Function

<img width="1280" height="853" alt="image" src="https://github.com/user-attachments/assets/f9fff620-33ad-4317-8631-ac6b7e2b908c" />
