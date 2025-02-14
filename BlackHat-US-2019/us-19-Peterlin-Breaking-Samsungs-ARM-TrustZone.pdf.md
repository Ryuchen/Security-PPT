1.1
BREAKING SAMSUNG'S ARM TRUSTZONE
Black Hat USA 2019

1.2
OUR TEAM
Maxime Peterlin - R&D Engineer at Quarkslab @pandasec_
Alexandre Adamski - R&D Engineer at Quarkslab @NeatMonster_
Joffrey Guilbon - R&D Engineer at Quarkslab @patateQBool

1.3
PRESENTATION OUTLINE
Current state of embedded security Introduction to the ARM TrustZone technology Samsung's TrustZone Overview Trusted Components Vulnerability Research Tools Vulnerability Analysis Exploitation Post-Exploitation Demonstrations

2.1
CURRENT STATE OF EMBEDDED SECURITY

2.2

A LONG TIME AGO...

TRADITIONAL ARCHITECTURE

APPLICATION

APPLICATION

APPLICATION

Kernel unbreakable...?

OPERATING SYSTEM

2.3
HOW DO WE PROTECT OURSELVES...
... if the kernel is corrupted during the boot process? ... if the kernel is corrupted when the system is already running?

2.4
PROTECTION DURING THE BOOT PROCESS
Secure Boot

STAGE 0 BOOTROM
VERIFIES, LOADS & EXECUTES
STAGE 1
...
STAGE N
VERIFIES, LOADS & EXECUTES
STAGE N+1
...

Prevent the execution of untrusted or unauthorized code on end users devices

2.5

RUNTIME PROTECTION USING AN HYPERVISOR

HYPERVISOR-BASED ARCHITECTURE

APP

APP

APP

APP

Hypervisor based guest kernel protection Problem: VM escapes and hypervisor compromissions

GUEST OS

GUEST OS

HYPERVISOR

2.6
TRUSTED EXECUTION ENVIRONMENTS Taken from Le TEE, nouvelle ligne de d�fense dans les mobiles, SSTIC 2013

Virtual Processor (e.g. ARM TrustZone)

On-SoC Processor (e.g. Apple SEP)

External Coprocessor (e.g. Google Titan M)

CPU

PERIPHERALS GPU CRYPTO HW OTP EEPROM RAM ROM

SoC

CPU
SECURE CPU

PERIPHERALS GPU CRYPTO HW OTP EEPROM RAM ROM
SoC

CPU

PERIPHERALS GPU CRYPTO HW OTP EEPROM RAM ROM

SoC
SECURE COPROC

3.1
ARM TRUSTZONE TECHNOLOGY

3.2
OVERVIEW
ARM TrustZone is a system-wide hardware isolation mechanism

CPU

PERIPHERALS GPU CRYPTO HW OTP EEPROM RAM ROM

SoC

Hardware architecture Partitioning of all the SoC's hardware and software resources TZPC, TZASC, TZMA, etc.
Software architecture Software implementation used in secure state Communications between secure and nonsecure components

3.3

SECURE AND NON-SECURE WORLDS

NORMAL WORLD
APPLICATIONS REQUIRING
APPLICATIONS TRUSTED
FUNCTIONALITY SUPPORT

EMBEDDED OS

TRUSTZONE DRIVERS

SECURE WORLD
SECURE OS MONITOR

TRUSTED APPLICATIONS
TRUSTED APPLICATIONS
TRUSTED APPLICATIONS

Secure World Runs trusted code Performs sensitive operations
Normal World Considered as compromised by design Performs non-sensitive operations

3.4

SECURE CONFIGURATION REGISTER

The Secure (or Non-Secure) state of the CPU is determined by the least signi cant bit of the Secure Con guration Register (SCR)

31

14 13 12 11 10 9 8 7 6 5 4 3 2 1 0

NS BIT [0]:

TWE TWI RES0 N SIF HCE SCD nET AW FW EA FIQ IRQ NS

RES0

0 SECURE STATE 1 NON-SECURE STATE

BITS ADDED IN AARCH64

3.5
COMMUNICATING BETWEEN WORLDS

NORMAL WORLD

SECURE WORLD

RICH OS

DATA

SMC INSTRUCTION INTERRUPT EXTERNAL ABORT

SECURE OS
DATA

SECURE MONITOR

NORMAL WORLD

SECURE WORLD

RICH OS

SECURE OS

DATA

DATA

SMC INSTRUCTION INTERRUPT EXTERNAL ABORT DIRECTLY WRITING TO THE CPSR REGISTER

SECURE MONITOR

Switches between worlds are performed by the Secure Monitor
Runs at the highest privilege level (EL3 in ARMv8/Monitor Mode in ARMv7)
Data exchanged through Exceptions Interruptions Writing to the PSTATE/CPSR registers (privileged operation)

3.6

PRIVILEGES SEPARATION

ARMv7 Privilege Levels

USER MODE (PL0)

NORMAL WORLD
APP APP APP

SECURE WORLD
APP APP APP

SUPERVISOR MODE (PL1)

GUEST OS

GUEST OS

SECURE OS

HYPERVISOR MODE (PL2)

HYPERVISOR

MONITOR MODE

SECURE MONITOR

ARMv8 Exception Levels

NORMAL WORLD

SECURE WORLD

EL0 APP APP APP APP APP APP

EL1 GUEST OS GUEST OS

SECURE OS

EL2

HYPERVISOR

EL3

SECURE MONITOR

3.7
THE TRUSTED COMPONENTS FRAGMENTATION ISSUE

NORMAL WORLD

USER MODE EL0

APP

APP

APP

SUPERVISOR MODE GUEST OS GUEST OS

EL1

TRUSTED OS DRIVERS

TRUSTED OS DRIVERS

HYPERVISOR MODE EL2

HYPERVISOR
TRUSTED OS DRIVERS

SECURE WORLD
APP APP APP
TRUSTED OS
TRUSTED HARDWARE DRIVERS

MONITOR MODE EL3

SECURE MONITOR

TRUSTED OS DISPATCHER

TRUSTED OS DISPATCHER

APPLICATION SPECIFIC GENERIC SOFTWARE SILICON PROVIDER SPECIFIC TRUSTED OS SPECIFIC SILICON PROVIDER SPECIFIC OR GENERIC SOFTWARE

Privilege escalation by design No hardware isolation between SEL1 and EL3 Access to all the physical memory Will be xed in ARMv8.4 with Secure Partitions
Fragmentation System developed by different vendors Cooperation and mutual trust required

3.8

TRUSTZONE SOFTWARE ARCHITECTURE
Several implementations of the software stack running in TrustZone are possible

Operating System

NORMAL WORLD
APPLICATIONS REQUIRING
APPLICATIONS TRUSTED
FUNCTIONALITY SUPPORT

SECURE WORLD

EMBEDDED OS

TRUSTZONE DRIVERS

SECURE OS MONITOR

Synchronous Library

NORMAL WORLD
APPLICATIONS REQUIRING
APPLICATIONS TRUSTED
FUNCTIONALITY SUPPORT

SECURE WORLD
MONITOR (SYNCHRONOUS LIBRARY)

EMBEDDED OS

TRUSTZONE DRIVERS

Intermediate Option

NORMAL WORLD
APPLICATIONS REQUIRING
APPLICATIONS TRUSTED
FUNCTIONALITY SUPPORT

SECURE WORLD

EMBEDDED OS

TRUSTZONE DRIVERS

MONITOR

TRUSTED APPLICATIONS
TRUSTED APPLICATIONS
TRUSTED APPLICATIONS
TRUSTED APPLICATIONS
TRUSTED APPLICATIONS
TRUSTED APPLICATIONS

3.9
TRUSTZONE'S USE CASES
Accessing hardware-backed features: Cryptographic engine Credentials storage (Hardware-backed Keystore) True random number generator ...
Digital Rights Management (by leveraging the cryptographic engine) Protecting and monitoring of the Normal World by the Secure World
Example: Samsung's Real-Time Kernel Protection (RKP) and Periodic Kernel Measurement (PKM)

4.1
SAMSUNG'S ARM TRUSTZONE

4.2
OVERVIEW
Samsung Devices Use both Samsung's Exynos and Qualcomm's Snapdragon SoCs The same phone models can have different SoCs depending on the country
Samsung's TrustZone Found only on Exynos SoCs First used in the Samsung Galaxy S3 Trusted OS used: Kinibi developed by Trustonic (Galaxy S3 to Galaxy S9) TEEGRIS developed by Samsung (Galaxy S10) Both are used in other models too This talk will focus on Kinibi

4.3
PREVIOUS WORKS
Reverse Engineering Samsung S6 SBOOT (2-part article series) by Fernand Lone Sang
ARM Trusted Firmware usage on Samsung devices and extraction process from an OTA of the TEE-OS
Unbox Your Phone (3-part article series) by Daniel Komaromy Reverse-engineering of the Trusted OS and exploitation of vulnerabilities in trustlets
Trust Issues: Exploiting TrustZone TEEs by Gal Beniamini Security analysis of different Trusted Execution Environments

4.4
SAMSUNG'S TRUSTZONE ARCHITECTURE

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

4.5
NORMAL WORLD COMPONENTS

NORMAL WORLD

APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

Drivers, daemons, libraries and interfaces used for communicating with the Secure World Communications pass through SMCs and shared memory buffers

4.6
SECURE MONITOR

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC

ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

ARM Trusted Firmware Open-source reference implementation of Secure World software provided by ARM Contains a modular secure monitor implementation Custom SMC handlers, called runtime services, can be added to t the vendors requirements Example: runtime services are used by Samsung to forward SMCs handled by Kinibi

4.7
SECURE WORLD COMPONENTS

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

Secure world based on a microkernel architecture

4.8
MTK: KINIBI'S MICRO KERNEL

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

Kinibi is a 32-bit OS developed by Trustonic Used to be called Mobicore and t-base
MTK: micro-kernel and only component running in S-EL1 Provides syscalls (SVCs)
Memory mapping, process creation, SMCs, etc. SVCs available depend on the privileges of the calling process Loads other components (embedded drivers, etc.) and especially RTM

4.9
RUN-TIME MANAGER

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

Special Secure World trusted application equivalent to the init process on Linux Main tasks
starting and managing processes notifying trustlets of incoming data from the NWd Implements communication channels Inter-Process Communications Mobicore Communication Interface (MCI)
A communication channel with the Normal World based on the Mobicore Control Protocol (MCP)

4 . 10
MCLIB: KINIBI'S STANDARD LIBRARY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

Provides standard functions to Trusted Applications, Secure Drivers and RTM Separated into two APIs:
TlApi: set of functions used by trusted applications DrApi: set of functions used by secure drivers Useful during exploitation to nd gadgets

TlApi call example

; _DWORD tlApiWaitNotification(_DWORD timeout)

MOV.W

R1, #0x1000

LDR.W

R2, [R1,#(tlApiLibEntry - 0x1000)]

MOV

R1, R0

MOVS

R0, #6

BX

R2

4 . 11
TRUSTED APPLICATIONS

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

Secure World equivalent of regular applications in the Normal World (run at SEL0) Allow trusted third-parties to extend the functionalities of the TEE-OS
Trusted UI, DRM, storage of secrets, etc. Signed binaries loaded directly from the Normal World (so are SDs)

4 . 12

TRUSTED APPLICATIONS LIFE-CYCLE

MAIN FUNCTION
STACK INITIALIZATION TCI BUFFER SIZE CHECK
...

tlApiWaitNotification

COMMAND #1 HANDLER

...

COMMAND #N HANDLER

tlApiNotify

Communications with the Normal World made through world-shared memory (named TCI buffer by Trustonic) The TCI buffer contains commands to be handled by the trustlet TCI buffer contains commands to be handled by the trustlet
Noti cations tlApiWaitNotification tlApiNotify

4 . 13
SECURE DRIVERS

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

Special type of Trusted Applications Run at S-EL0 but have higher software-de ne privileges Have access to a richer set of API and syscalls Are used by trustlets as an interface to access physical memory and reach secure peripherals in a controlled manner Communications with TAs made through IPCs and shared memory

DCI HANDLER drApiIpcSigWait
... drApiNotify

4 . 14

SECURE DRIVERS LIFE-CYCLE

ENTRY POINT
MAIN THREAD
IPC HANDLER drApiIpcCallToIPCH
...

ISR HANDLER drApiIntrAttach
... drApiIntrDetach

Multi-threaded application DCI: Normal World communications IPC: trustlet communications
Trustlet interactions Retrieves IPC data by mapping the entire trustlet Noti cations using drApiIpcCallToIPCH

5.1
VULNERABILITY RESEARCH TOOLS

5.2
ATTACK SURFACE

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE (TCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

MOBICORE COMMUNICATION INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)
SYSCALL HANDLERS KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

5.3
ATTACK SURFACE

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE (TCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

MOBICORE COMMUNICATION INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)
SYSCALL HANDLERS KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

5.4
ATTACK SURFACE

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE (TCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

MOBICORE COMMUNICATION INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

DRCRYPT
...
RUNTIME MANAGEMENT (RTM)
SVC
SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

5.5
ATTACK SURFACE

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE (TCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

MOBICORE COMMUNICATION INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)
SYSCALL HANDLERS KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

SMC
TEE-OS DISPATCHER

5.6
ATTACK SURFACE

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE (TCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

MOBICORE COMMUNICATION INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)
SYSCALL HANDLERS KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

5.7
ATTACK SURFACE
Must be reacheable from the Normal World ATF is open-source, probably heavily reviewed Trusted Applications are low-hanging fruits

5.8
OUR JOURNEY IN 5 STEPS

5.9
STEP #1 - LOADING INTO IDA/GHIDRA
LOADER

5 . 10
Proprietary File Format - MobiCore Loadable Format (MCLF)

5 . 11

mclf-ida-loader

mclf-ghidra-loader

5 . 12
STEP #2 - IDENTIFYING FUNCTIONS
LOADER SCRIPTS

5 . 13
MCLIB - STANDARD LIBRARY

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE (TCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

MOBICORE COMMUNICATION INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)
SYSCALL HANDLERS KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

5 . 14
Renames tlApi/drAPI functions Sets the functions prototypes

Before

After

5 . 15

STEP #3 - MANUALLY FINDING VULNERABILITIES

LOADER

EMULATOR

SCRIPTS

5 . 16
TRUSTLETS EMULATOR
Based on Unicorn (external project) Split into simple tasks:
Loading the MCLF binary Mapping the shared memory buffer Hooking the McLib functions

5 . 17
TRUSTLETS EMULATOR
python emulator.py *41.tlbin cmd1.bin --tci 0x40100 -v [+] Binary is a trustlet [+] Trustlet size = 0x1ba4c [+] Mapping text section at 0x00001000 with a size of 0x4874 [+] Mapping data section at 0x00007000 with a size of 0x168 [+] Mapping BSS section at 0x00007168 with a size of 0x17070 [+] Mapping region at 0x07d00000 (0x1 bytes) [+] Mapping TCI buffer at 0x00100000 with a size of 0x40100 [i] drApiLogvPrintf(u'ICCC:Trustlet ICCC::Starting\n') [+] Loading input data [i] drApiLogvPrintf(u'TL ICCC: we got a command: 1\n') [i] drApiLogvPrintf(u'ICCC: Initialize failed - tamper fuse set\n') [i] drApiLogvPrintf(u'ICCC: Measurements result ret = 65548, ret hex = 1000c\n') [i] drApiLogvPrintf(u'iccc: ICCC save data@#\n') [i] drApiLogvPrintf(u'Iccc_phys_read failed\n') [i] drApiLogvPrintf(u'ICCC: check magic failed\n') [i] drApiLogvPrintf(u'End of ICCC_Init, ret=1000c\n') [i] drApiLogvPrintf(u'ICCC: Error writing Trustboot flag\n') [+] tlApiNotify: Quitting!

5 . 18

STEP #4 - FINDING VULNERABILITIES AUTOMATICALLY

LOADER

EMULATOR

SCRIPTS

FUZZER

5 . 19
TRUSTLETS FUZZER
Based on AFL_Unicorn (internal project) Interfaces the fuzzer AFL with Unicorn Usability and performance improvements 100% of the code is written in Python!

5 . 20
TRUSTLETS FUZZER

5 . 21
TRUSTLETS SYMBOLIC EXECUTOR
Based on Manticore by Trail of Bits Uses very simple strategies:
Mark the shared memory buffer symbolic Explore all the paths of the trustlet Check reads or writes to memory Ask the solver for an invalid address

5 . 22

CRASH EXAMPLE

Command line: '../tainter.py -s 1036 ffffffff000000000000000000000005.tlbin.elf -t -c coverage.txt'
Status: Invalid symbolic memory access (mode:r)

================ PROC: 00 ================

Memory:

0000000000001000-0000000000008000 r x 00000094 ffffffff000000000000000000000005.tlbin.elf

0000000000009000-0000000000024000 rw 00006dc8 ffffffff000000000000000000000005.tlbin.elf

0000000000100000-0000000000101000 rw 00000000

0000000007d00000-0000000007d01000 rx 00000000 CPU:

INSTRUCTION: 0x00000000000059ec:

pld [r1, #0x80]

APSR: 0x0000000060000000

R0 : 0x0000000000009aac

R1 : <BitVecExtract at 7f2571dbdeb8-T>

R10: 0x0000000000000000

R11: 0x0000000000000000

R12: 0x0000000000000000

R13: 0x0000000000023a28

5 . 23

STEP #5 - EXPLOITING THE VULNERABILITIES

LOADER

EMULATOR

BINDINGS

SCRIPTS

FUZZER

5 . 24
SOFTWARE STACK

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE (TCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

MOBICORE COMMUNICATION INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)
SYSCALL HANDLERS KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

5 . 25
CLIENT API

CONNECTOR
allocate MCOPEN SESSION WRITE TO MCNOTIFY
MCWAIT NOTIFICATION READ FROM
MCCLOSE SESSION
FREE

WORLD SHARED MEMORY BUFFER

TRUSTLET
TLAPIWAIT NOTIFICATION READ FROM
WRITE TO TLAPINOTIFY

5 . 26
PYTHON BINDINGS
Writing C is tedious, writing Python is a lot easier Bindings of the mcClient API called pymcclient Provides various utilities: hexdump, (dis)assemble, etc. Provides a command interpreter which is based on IPython

5 . 27
SCRIPT EXAMPLE
with Device(DEVICE_ID) as dev: with dev.buffer(TCI_BUFFER_SIZE) as tci: with open(TRUSTLET_FILE, "rb") as fd: buf = fd.read() with Trustlet(dev, tci, buf) as app: tci.seek(0) tci.write_dword(1) app.notify() app.wait_notification() tci.seek(0) print(tci.read_dword())

6.1
VULNERABILITY ANALYSIS & EXPLOITATION

6.2
OVERVIEW
Target: Samsung Galaxy S7 running Android 7.0
Main goal: Obtaining code execution in EL3
Prerequisites: Being part of the radio group Being able to write les somewhere on the device

6.3
ATTACK PLAN

NORMAL WORLD

APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

HERE!
TEE-OS DISPATCHER

6.4

SOFTWARE MITIGATIONS

Model XN bit Canary ASLR PIE

S6









S7









S8









S9









6.5

ATTACKING A TRUSTED APPLICATION
Overview

NORMAL WORLD

APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

WE'RE HERE!
TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

6.6

ATTACKING A TRUSTED APPLICATION
SEM Trustlet Vulnerability

MAIN FUNCTION
STACK INITIALIZATION TCI BUFFER SIZE CHECK
...

tlApiWaitNotification

COMMAND

COMMAND

COMMAND

#1

...

#5

...

#N

HANDLER

HANDLER

HANDLER

tlApiNotify

Stack-based buffer over ow in the handler of the command ID #5

Call to memcpy at the beginning of the 5th command handler

Before this call, the registers are set as follow:

R0 = SP+0x4F8-0xF0, the destination buffer R1 = tci_buffer + 0x8, the source buffer R2 = *(tci_buffer + 0x16808), the length of the buffer

.text:00020FB2 .text:00020FB6 .text:00020FB8 .text:00020FBC .text:00020FC0 .text:00020FC4

ADD.W MOV LDR.W ADD.W ADD.W BLX

R1, R0, #0x16000 R4, R1 R2, [R1,#0x808] R1, R0, #8 R0, SP, #0x4F8+var_F0 memcpy_aligned

6.7

ATTACKING A TRUSTED APPLICATION
Exploitation Results

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM CONTAINERS
HERE!
SYSTEM TA

SECURE DRIVERS
DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

Code execution in S-EL0 It is now possible to:
Communicate with Secure Drivers Make some syscalls (e.g. print characters, get system information, etc.) Next target: Secure Driver

6.8

ATTACKING A SECURE DRIVER
VALIDATOR Secure Driver Vulnerability

A vulnerability was found in the VALIDATOR secure driver

Stack-based buffer over ow in the handler of the command ID #15

Equivalent to the one found in the trustlet (i.e. memcpy in the stack and a user-controlled size)

.text:00001362 .text:00001364 .text:00001366 .text:00001368

MOVS MOV ADDS BLX

R2, #0x37 ; '7' R1, R4 R0, R6, #1 memcpy

6.9

ATTACKING A SECURE DRIVER
Exploitation Results

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM CONTAINERS
SYSTEM TA

SECURE DRIVERS
HERE!
DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

TEE-OS DISPATCHER

Code execution in S-EL0 but with higher privileges It is now possible to:
Communicate with the RunTime Manager (or RTM, an init-like process) Access more syscalls (e.g. map physical memory, create threads, make SMCs, etc.) Next target: Trusted OS & Monitor

6 . 10
ATTACKING KINIBI AND THE MONITOR
Vulnerability Analysis mmap: secure and non-secure physical memory mapping syscall Vulnerability
Monitor mapped at 0x2022000 Can be mapped using mmap to modify an SMC Calling the hijacked SMC allows code execution in EL3 Patch Fixed in the newest versions by using a blacklist

6 . 11

ATTACKING KINIBI AND THE MONITOR
Exploitation Results

NORMAL WORLD APP

SERVICE PROVIDER PROVISIONING
AGENT
ROOT PROVISIONING
AGENT

TRUSTED APPLICATION CONNECTOR

KINIBI CLIENT API

KINIBI DAEMON

KINIBI EMBEDDED KERNEL

DRIVERS

OS

TRUSTED APPLICATION COMMUNICATION INTERFACE
(TCI)
SHARED NON-
SECURE MEMORY
MOBICORE COMMUNICATION
INTERFACE (MCI)
SHARED NON-
SECURE MEMORY

SECURE WORLD

SiP/OEM SYSTEM SECURE DRIVERS CONTAINERS

SYSTEM TA

DRIVER TA

TLAPI MCLIB DRAPI

DRCRYPT ...
RUNTIME MANAGEMENT (RTM)

SYSCALL HANDLERS
KINIBI MICRO-KERNEL

SMC
ATF

MONITOR (ATF BL31)
FORWARD TO CUSTOM TEE-OS HANDLER

HERE!
TEE-OS DISPATCHER

Code execution in EL3 Now possible to do anything we want!

7.1
POST-EXPLOITATION

7.2
TRUSTPWN FRAMEWORK
Based on the previous vulnerabilities Internals
Uses the EL3 vulnerability to have arbitrary access to Kinibi Adds a SVC and a drApi function to execute code in S-EL1 "natively"
SVCs and drApi functions are referenced in pointer arrays Usage Read or write memory arbitrarily Execute code in S-EL1 and EL3

7.3
DEMO
Finding the Master Key in the Monitor

7.4
FINDING THE MASTER KEY IN THE MONITOR
DrApi Reversing
Reversing the crypto-driver drcrypto (found embedded in Kinibi)
DrApi 0x1030 Takes four possible command IDs (0xAA, 0xAB, 0xAC, 0xAD) The interesting one is 0xAB Wrapper around SMC 0xB2000005
SMC 0xB2000005 SMC arguments: R0: SMC ID R0: command number (four possible values [0-3]) R1: number of bytes to read Reads 0x10 bytes of the master key at 0x101E4000

7.5
FINDING THE MASTER KEY IN THE MONITOR
DrApi Function

7.6
FINDING THE MASTER KEY IN THE MONITOR
SMC Function

7.7
DEMO
Bypassing Signature Checks

7.8
BYPASSING SIGNATURE CHECKS
Methodology Reversing RTM Finding the SHA-256 of the public key corresponding to the private key used to sign TAs and SDs Signature is veri ed using tlApiSignatureVerify Patch the checks and load your own TA or SD

7.9

FINDING THE MASTER KEY IN THE MONITOR
RTM Veri cations

First check

ROM:00006E62 ROM:00006E66 ROM:00006E68 ROM:00006E6A ROM:00006E6C

BL LDR ADDS CBNZ LDRB.W

tlApiSignatureVerify R4, =0x40B00009 R4, R4, #6 R0, loc_6E7A R0, [SP,#0xC0+var_44]

Second check

ROM:000073E0 ROM:000073E4 ROM:000073E6

BL CBNZ LDRB.W

tlApiSignatureVerify R0, loc_73FA R0, [SP,#0x1C0+var_48]

7 . 10
DEMO
Trusted-OS Instrumentation

7 . 11
TRUSTED-OS INSTRUMENTATION
Methodology Handles ARMv7 and Thumb Based on the Unde ned Instruction exception Unde ned Instruction handler is replaced by our own code Patch an instruction with the ARM unde ned instruction UDF 0xNNNN When a breakpoint triggers the current context of the CPU is saved
Current context is saved Overwritten instruction is executed

8.1
THANK YOU!

