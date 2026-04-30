---
title: "LLVM Bootstrap Install"
description: "Install LLVM by bootstrapping"
date: 2026-02-24
draft: false
tags: ["LLVM"]
categories: ["LLVM"]
---

## Why need to bootstrap LLVM
For multi-platform installation, due to different ABI of libstdc++(excluding libc if need musl instead) in its kernel, it's hard to promise a successful installation. So using libc++abi/libc++ AKA libcxxabi/libcxx highly integrated in LLVM is reasonable.

Once LLVM built in a unified ABI, it's more easier to avoid the differentiation between different platforms.

## LLVM Bootstrap Install

Such on ArchLinux, Clang, LLVM, MLIR(through yay, but self-compiled) and other llvm subprojects installed by `pacman -S clang llvm #if more subprojects available to install`, all of them are prebuilt based on libstdc++ AKA libstdcxx, like:
```bash
pacman -Qi clang
Name            : clang
Version         : 21.1.8-1
Description     : C language family frontend for LLVM
Architecture    : x86_64
URL             : https://clang.llvm.org/
Licenses        : Apache-2.0 WITH LLVM-exception
Groups          : None
Provides        : clang-analyzer=21.1.8  clang-tools-extra=21.1.8
Depends On      : llvm-libs  gcc  compiler-rt
Optional Deps   : openmp: OpenMP support in clang with -fopenmp [installed]
                  python: for scan-view and git-clang-format [installed]
                  llvm: referenced by some clang headers [installed]
Required By     : doxygen  kdevelop  lldb
Optional For    : kate  qt6-tools
Conflicts With  : clang-analyzer  clang-tools-extra
Replaces        : clang-analyzer  clang-tools-extra
Installed Size  : 217.27 MiB
Packager        : Leonidas Spyropoulos <artafinde@archlinux.org>
Build Date      : Mon 02 Feb 2026 08:42:29 PM CST
Install Date    : Sun 15 Feb 2026 10:59:24 AM CST
Install Reason  : Explicitly installed
Install Script  : No
Validated By    : Signature
ldd /usr/bin/clang
        linux-vdso.so.1 (0x00007fdcb69ec000)
        libclang-cpp.so.21.1 => /usr/lib/libclang-cpp.so.21.1 (0x00007fdcb2e00000)
        libLLVM.so.21.1 => /usr/lib/libLLVM.so.21.1 (0x00007fdca9600000)
        libstdc++.so.6 => /usr/lib/libstdc++.so.6 (0x00007fdca9200000) # libstdc++ here
        libc.so.6 => /usr/lib/libc.so.6 (0x00007fdca900f000)
        libm.so.6 => /usr/lib/libm.so.6 (0x00007fdcb2ce2000)
        libgcc_s.so.1 => /usr/lib/libgcc_s.so.1 (0x00007fdcb2cb5000)
        /lib64/ld-linux-x86-64.so.2 => /usr/lib64/ld-linux-x86-64.so.2 (0x00007fdcb69ee000)
        libffi.so.8 => /usr/lib/libffi.so.8 (0x00007fdcb696e000)
        libedit.so.0 => /usr/lib/libedit.so.0 (0x00007fdca95c6000)
        libz.so.1 => /usr/lib/libz.so.1 (0x00007fdcb2c9a000)
        libzstd.so.1 => /usr/lib/libzstd.so.1 (0x00007fdca94e0000)
        libxml2.so.16 => /usr/lib/libxml2.so.16 (0x00007fdca8eda000)
        libncursesw.so.6 => /usr/lib/libncursesw.so.6 (0x00007fdca8e69000)
        libicuuc.so.78 => /usr/lib/libicuuc.so.78 (0x00007fdca8c00000)
        libicudata.so.78 => /usr/lib/libicudata.so.78 (0x00007fdca6c00000)
```

### Stage0
```bash
cmake -G Ninja -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm-stage0 \
    -DLLVM_ENABLE_PROJECTS="clang" \
    -DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind;compiler-rt" \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    ../llvm
```

### Stage1
```bash
cmake -G Ninja -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_INSTALL_PREFIX=/opt/llvm-custom \
    -DLLVM_ENABLE_PROJECTS="clang;lld;clang-tools-extra;mlir" \
    -DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind;compiler-rt" \
    -DCMAKE_C_COMPILER=/opt/llvm-stage0/bin/clang \
    -DCMAKE_CXX_COMPILER=/opt/llvm-stage0/bin/clang++ \
    -DCMAKE_CXX_FLAGS="-stdlib=libc++ -isystem /opt/llvm-stage0/include/c++/v1" \
    -DCMAKE_EXE_LINKER_FLAGS="-L/opt/llvm-stage0/lib -L/opt/llvm-stage0/lib/x86_64-unknown-linux-gnu -Wl,-rpath,/opt/llvm-stage0/lib -Wl,-rpath,/opt/llvm-stage0/lib/x86_64-unknown-linux-gnu" \
    -DCMAKE_SHARED_LINKER_FLAGS="-L/opt/llvm-stage0/lib -L/opt/llvm-stage0/lib/x86_64-unknown-linux-gnu -Wl,-rpath,/opt/llvm-stage0/lib -Wl,-rpath,/opt/llvm-stage0/lib/x86_64-unknown-linux-gnu" \
    -DLLVM_ENABLE_LLD=ON \
    -DCLANG_DEFAULT_CXX_STDLIB=libc++ \
    -DCLANG_DEFAULT_RTLIB=compiler-rt \
    -DLLVM_TARGETS_TO_BUILD="X86" \
    -DCMAKE_BUILD_RPATH="/opt/llvm-stage0/lib;/opt/llvm-stage0/lib/x86_64-unknown-linux-gnu" \
    -DCMAKE_INSTALL_RPATH="/opt/llvm-custom/lib;/opt/llvm-custom/lib/x86_64-unknown-linux-gnu" \
    -DCMAKE_INSTALL_RPATH_USE_LINK_PATH=ON \
    ../llvm
```


### Stage2
```bash
cmake -G Ninja -DCMAKE_BUILD_TYPE=Release \
	-DCMAKE_INSTALL_PREFIX=/opt/llvm-base \
	-DLLVM_ENABLE_PROJECTS="clang;lld;clang-tools-extra;mlir" \
	-DLLVM_ENABLE_RUNTIMES="libcxx;libcxxabi;libunwind;compiler-rt" \
	-DCMAKE_C_COMPILER=/opt/llvm-custom/bin/clang \
	-DCMAKE_CXX_COMPILER=/opt/llvm-custom/bin/clang++ \
	-DCMAKE_CXX_FLAGS="-stdlib=libc++ -isystem /opt/llvm-custom/include/c++/v1" \
	-DCMAKE_EXE_LINKER_FLAGS="-L/opt/llvm-custom/lib -L/opt/llvm-custom/lib/x86_64-unknown-linux-gnu -Wl,-rpath,/opt/llvm-custom/lib -Wl,-rpath,/opt/llvm-custom/lib/x86_64-unknown-linux-gnu" \
	-DCMAKE_SHARED_LINKER_FLAGS="-L/opt/llvm-custom/lib -L/opt/llvm-custom/lib/x86_64-unknown-linux-gnu -Wl,-rpath,/opt/llvm-custom/lib -Wl,-rpath,/opt/llvm-custom/lib/x86_64-unknown-linux-gnu" \
	-DLLVM_ENABLE_LLD=ON -DCLANG_DEFAULT_CXX_STDLIB=libc++ \
	-DCLANG_DEFAULT_RTLIB=compiler-rt -DLLVM_TARGETS_TO_BUILD="X86" \
    	-DCMAKE_BUILD_RPATH="/opt/llvm-custom/lib;/opt/llvm-custom/lib/x86_64-unknown-linux-gnu" \
    	-DCMAKE_INSTALL_RPATH="/opt/llvm-base/lib;/opt/llvm-base/lib/x86_64-unknown-linux-gnu" \
    	-DCMAKE_INSTALL_RPATH_USE_LINK_PATH=ON \
	../llvm
```


### Stage3
```bash
cmake -G Ninja -DCMAKE_BUILD_TYPE=Release \
	-DCMAKE_INSTALL_PREFIX=/opt/llvm-base \
	-DLLVM_ENABLE_PROJECTS="clang;lld;clang-tools-extra;mlir" \
	-DCMAKE_C_COMPILER=/opt/llvm-base/bin/clang \
	-DCMAKE_CXX_COMPILER=/opt/llvm-base/bin/clang++ \
	-DCMAKE_CXX_FLAGS="-stdlib=libc++ -isystem /opt/llvm-base/include/c++/v1" \
	-DCMAKE_EXE_LINKER_FLAGS="-L/opt/llvm-base/lib -L/opt/llvm-base/lib/x86_64-unknown-linux-gnu -Wl,-rpath,/opt/llvm-base/lib -Wl,-rpath,/opt/llvm-base/lib/x86_64-unknown-linux-gnu" \
	-DCMAKE_SHARED_LINKER_FLAGS="-L/opt/llvm-base/lib -L/opt/llvm-base/lib/x86_64-unknown-linux-gnu -Wl,-rpath,/opt/llvm-base/lib -Wl,-rpath,/opt/llvm-base/lib/x86_64-unknown-linux-gnu" \
	-DLLVM_ENABLE_LLD=ON -DCLANG_DEFAULT_CXX_STDLIB=libc++ \
	-DCLANG_DEFAULT_RTLIB=compiler-rt -DLLVM_TARGETS_TO_BUILD="X86" \
    	-DCMAKE_BUILD_RPATH="/opt/llvm-base/lib;/opt/llvm-base/lib/x86_64-unknown-linux-gnu" \
    	-DCMAKE_INSTALL_RPATH="/opt/llvm-base/lib;/opt/llvm-base/lib/x86_64-unknown-linux-gnu" \
    	-DCMAKE_INSTALL_RPATH_USE_LINK_PATH=ON \
    	-DLLVM_BUILD_LLVM_DYLIB=ON \
    	-DLLVM_LINK_LLVM_DYLIB=ON \
    	-DCLANG_LINK_CLANG_DYLIB=ON \
    	-DLLVM_DYLIB_COMPONENTS="all" \
    	-DLLVM_ENABLE_LIBEDIT=OFF \
    	-DLLVM_ENABLE_ZLIB=OFF \
    	-DLLVM_ENABLE_ZSTD=OFF \
    	-DLLVM_ENABLE_LIBXML2=OFF \
    	-DLLVM_ENABLE_RTTI=ON \
	-DLLVM_INSTALL_UTILS=ON \
	../llvm
```
