# toolchain-build

## About this fork

This is a fork of [c-sky/toolchain-build](https://github.com/c-sky/toolchain-build)
by C-SKY Microsystems (Hangzhou C-SKY Microsystems Co., Ltd., now part of T-Head / XuanTie).
The `gcc-6_3-build` branch is based on upstream commit
[`b849ba3`](https://github.com/c-sky/toolchain-build/commit/b849ba315b998384f2046328b0416efddc6b1a86)
("Add 860 multilib", 2018-04-21), the original single-script layout that builds the
GCC 6.3-based C-SKY toolchain. Upstream `master` has since been restructured and is not
carried here.

**Upstream authors** (from the upstream git history): quxianmiao / quxm (Xianmiao Qu, C-SKY)
wrote `build-csky-gcc.py`, the examples and this README; later upstream work is by quxm and
Cooper Qu. The files under `examples/` carry their own
"Copyright (C) Hangzhou C-SKY Microsystem Co., Ltd." notices. `elf_minilibc_base.tar.gz` is an
upstream-supplied archive of prebuilt C-SKY libraries and headers and is shipped unmodified.

**Licensing.** The upstream repository has no license file and the script has no license
header. This fork grants no license for the upstream code (including `examples/` and
`elf_minilibc_base.tar.gz`); publishing it does not change the upstream authors' rights.
The license for this fork's own changes is to be decided.

**Changes on this branch** (all to `build-csky-gcc.py`), made so that the elf/minilibc C
toolchain (`--abi=abiv2 --tos=elf`) builds unattended inside a container on a current host:

- Port the script to Python 3.
- Clone binutils, gcc, minilibc and gdb over HTTPS instead of SSH, so no SSH key is needed.
- Default `--user` to an empty string when `$USER` is unset instead of crashing.
- Fix four invalid-escape `SyntaxWarning`s.
- Decode subprocess output to `str` in `cmd_exec`.
- Do not pass `--with-headers` to the stage-2 elf GCC build: the bundled minilibc headers lack
  `pid_t`, which broke libgcc.
- Build C only. libstdc++'s configure link tests fail under this cross build.
- Leave gdb out of the default component chain. The gdb-7.12-era sources do not compile with a
  current host C++ compiler; `-m gdb` still builds it on request.

The upstream build instructions below are unchanged.

## BUILD toolchain
First, create a directory for building toolchain.<br>
Second, enter the direcotry, and copy the script “build-csky-gcc.py” to this directory.<br>
C-SKY abi-v2 supports three versions of the toolchain to compile.<br>
### uclibc-ng
* Run the following command to download the required source code:<br>
`./build-csky-gcc.py --abi=abiv2 --tos=linux --libc=uclibc-ng --init`<br>
The code will be downloaded to the "source" directory under the current directory, you can also create your own "source" directory, and download the source code of the required components to the source directory<br>
* Execute the following command to compile the toolchain:<br>
`./build-csky-gcc.py --abi=abiv2 --tos=linux --libc=uclibc-ng`<br>
After the compilation is completed, the toolchain will be installed into the "install-csky-linux-uclibcabiv2-uclibc-ng" directory<br>
### glibc
* Run the following command to download the required source code:<br>
`./build-csky-gcc.py --abi=abiv2 --tos=linux --libc=glibc --init`<br>
* Execute the following command to compile the toolchain:<br>
`./build-csky-gcc.py --abi=abiv2 --tos=linux --libc=glibc`<br>

After the compilation is completed, the toolchain will be installed into the "install-csky-linux-gnuabiv2-glibc" directory<br>
### elf
* Run the following command to download the required source code:<br>
`./build-csky-gcc.py --abi=abiv2 --tos=elf --init`<br>
* Execute the following command to compile the toolchain:<br>
`./build-csky-gcc.py --abi=abiv2 --tos=elf`<br>
After the compilation is completed, the toolchain will be installed into the "install-csky-elfabiv2-minilibc" directory<br>
<br>
As mentioned above, you can use the script to compile all the toolchains, if you want to compile by yourself, you can add the "--fake" option to the build command to print out all the configuration and compile commands, for example:<br>

`./build-csky-gcc.py --abi=abiv2 --tos=linux --libc=uclibc-ng --fake`<br>



## BUILD qemu
* First, download the code:<br>
`git clone git@github.com:c-sky/qemu.git`<br>
* Second, create a directory for building, enter the directory, configure and build, for example:<br>
`mkdir qemu_build`<br>
`cd qemu_build`<br>
`${QEMU_SOURCE_DIR}/configure --target-list="cskyv2-softmmu cskyv2-linux-user cskyv2eb-softmmu cskyv2eb-linux-user" --prefix=${QEMU_INSTALL_DIR}`<br>
`make; make install`<br>
* Now, Qemu is now installed, you can enter the installation directory to view the generated program. In the ${QEMU_INSTALL_DIR}/bin directory, you can see the following programs:
>qemu-system-cskyv2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--csky little-endian qemu system mode executive program<br>
>qemu-system-cskyv2eb&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--csky big-endian qemu system mode executive program<br>
>qemu-cskyv2&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--csky little-endian qemu user mode executive program<br>
>qemu-cskyv2eb&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;--csky big-endian qemu user mode executive program<br>
  
  

## Runing examples for test
There are three examples in the "examples" directory.<br>
### smartcard_uart
This example can used for ck801/ck802/ck803 elf, add the compiled toolchain and qemu program directory to the current PATH environment variable, execute the following command to run this demo:<br>
`export PATH=${QEMU_INSTALL_DIR}/bin:${TOOLCHAIN_INSTALL_DIR}/bin:$PATH`<br>
`make clean`<br>
`make CPU=ck801/ck802/ck803`<br>
`qemu-system-cskyv2  -machine smart_card -cpu ck801/ck802/ck803 -nographic -kernel Uart.elf`<br>

You can see the output:<br>
>Testing uart...<br>
>Default configure: Baudrate --- 19200,Parity --- NONE,Wordsize --- 8. <br>
>- - -UART0 ready? [y]<br>

See this shows that the demo has been successfully executed, you can enter the characters interact with the program, or you can exit the emulator using: \<Ctrl-a x\>.
### trilobite_uart
This example can used for ck807/ck810 elf, add the compiled toolchain and qemu program directory to the current PATH environment variable, execute the following command to run this demo:<br>
`make clean`<br>
`make CPU=ck807/ck810`<br>
`qemu-system-cskyv2  -machine smart_card -cpu ck807/ck810 -nographic -kernel Uart.elf`<br>

You can see the output:<br>
>Testing uart...<br>
>Default configure: Baudrate --- 19200,Parity --- NONE,Wordsize --- 8. <br>
>- - -UART0 ready? [y]<br>
### linux_hello
This example can used for ck807/ck810 linux, add the compiled toolchain and qemu program directory to the current PATH environment variable, execute the following command to run this demo:<br>
`csky-abiv2-linux-gcc -mcpu=ck807/ck810 hello.c  -o hello --static`<br>
`qemu-cskyv2 -cpu ck807/ck810 hello`<br>

You can see the output:<br>
>"Hello World!"<br>


## The relationship between compile options and qemu options
Under normal circumstances, qemu cpu options only need to be consistent with the compiled cpu options.For example, when we use -mcpu=ck803e for compiling, just add -cpu ck803e for qemu.<br>
There are two options that require special attention.<br>
### -mbig-endian/-mlittle-endian
when used -mlittle-endian, execute the qemu-system-cskyv2 or qemu-cskyv2<br>
when used -mbig-endian, execute the qemu-system-cskyv2eb or qemu-cskyv2eb<br>
### -mhard-float/-msoft-float
when used -msoft-float, the cpu option for qemu does not need to add 'f'<br>
when used -mhard-float,the cpu option for qemu needs to add 'f'<br>
  
