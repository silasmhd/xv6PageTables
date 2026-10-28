# Page Table Enhancements for xv6

## Description
This project enhances xv6 page tables to speed up system calls via a shared vsyscall page, print page table contents for debugging, and detect accessed pages via RISC-V PTE access bits.

## Objectives
- Map a read-only vsyscall page at USYSCALL for fast `getpid()` without kernel crossing
- Implement `vmprint()` to visualize page table structure
- Implement `pgaccess()` system call to report accessed pages via PTE_A bit

## Implementation
- **USYSCALL mapping** — In `allocproc()`/`proc_pagetable()`, allocate and map a read-only page at USYSCALL with PID; free in `freeproc()`
- **vmprint(pagetable)** — Recursively walk page table, print valid PTEs with index, PTE bits, and physical address indented by depth
- **pgaccess(va, npages, buf)** — Scan `npages` starting at `va`, check/clear PTE_A bit in each PTE, write bitmask to user buffer via `copyout()`

## Build and Run Instructions

To build and run xv6, you need a RISC-V toolchain and QEMU emulator

I used WSL and ran Ubuntu 24.04 specifically, so there may be differences with different distributions and versions. Once you have wsl installed this will install the version of Ubuntu I used, but removing "-24.04" will install the latest.

```bash
wsl.exe --install Ubuntu-24.04
```

Next update and install the packages I used

```bash
sudo apt-get update && sudo apt-get upgrade

sudo apt-get install git build-essential qemu-system-misc gcc-riscv64-linux-gnu binutils-riscv64-linux-gnu  
```

Lastly clone this repository, change directory into it, and run QEMU.

```bash
git clone https://github.com/silasmhd/pageTables

cd pageTables

make qemu
```

To exit QEMU press `ctrl + a` then `x`. 
_This won't work with capslock on._

## Tests

Once you have QEMU running, you can run the test for this project and look at the them in `test-pgtbl`.

```bash
make test
```

## Files Modified
- `kernel/proc.c` — `allocproc()`, `freeproc()`, `proc_pagetable()` for USYSCALL page
- `kernel/vm.c` — `vmprint()`, `walk()`, `mappages()` helpers
- `kernel/memlayout.h` — USYSCALL address, `struct usyscall`, PTE_A bit definition
- `kernel/riscv.h` — PTE_A macro for access bit
- `kernel/sysproc.c` — `sys_pgaccess()` implementation
- `kernel/defs.h` — `vmprint()` prototype
- `kernel/kalloc.c` — page allocation for USYSCALL page
- `user/usys.pl` — `pgaccess` syscall stub
- `user/user.h` — `pgaccess`, `vmprint` prototypes

## References
- xv6 book, Chapter 3: Page Tables
- [RISC-V privileged architecture manual](https://github.com/riscv/riscv-isa-manual/releases/download/Ratified-IMFDQC-and-Priv-v1.11/riscv-privileged-20190608.pdf)
- RISC-V Privileged Architecture Specification (Sv32 page table format)
- Original xv6-riscv repository: https://github.com/mit-pdos/xv6-riscv
