# macOS-Control-Bypasses

## Table of Contents

1. Introduction to macOS
  - macOS Penetration Testing
  - High-Level OS Architecture
  - The Mach-O File Format
  - Objective-C Primer
  - Wrapping Up   
2. macOS Binary Analysis Tools
  - Command Line Static Analysis Tools
  - Static Analysis with Hopper
  - Dynamic Analysis
  - The LLDB Debugger
  - Debugging with Hopper
  - Tracing Applications with DTrace
  - Wrapping Up  
3. The Art of Crafting Shellcodes
  - Writing Shellcode in ASM
  - Custom Shell Command Execution in Assembly
  - Making a Bind Shell in Assembly
  - Writing Shellcode in C
  - Wrapping Up
4. Dylib Injection
  - DYLD_INSERT_LIBRARIES Injection in macOS
  - DYLIB Hijacking
  - Wrapping Up
5. The Mach Microkernel
  - Mach Inter Process Communication (IPC) Concepts
  - Mach Special Ports
  - Injection via Mach Task Ports
  - BlockBlock Case Study - Injecting execv Shellcode
  - Injecting a Dylib
  - Wrapping Up
6. Function Hooking on macOS
  - Function Interposing
  - Objective-C Method Swizzling
  - Wrapping Up
7. XPC Attacks
  - About XPC
  - The Low Level C API: XPC Services
  - The Foundation Framework API
  - Attacking XPC Services
  - Apple’s EvenBetterAuthorizationSample
  - CVE-2019-20057 - Proxyman Change Proxy Privileged Action Vulnerability
  - CVE-2020-0984 - Microsoft Auto Update Privilege Escalation Vulnerability
  - CVE-2019-8805 - Apple EndpointSecurity Framework Local Privilege Escalation
  - CVE-2020-9714 - Adobe Reader Update Local Privilege Escalation
  - Wrapping Up
8. The macOS Sandbox
  - Sandbox Internals
  - The Sandbox Profile Language (SBPL)
  - Sandbox Escapes
  - Case Study: QuickLook Plugin SB Escape
  - Case Study: Microsoft Word Sandbox Escape
  - Wrapping Up
9. Bypassing Transparency, Consent, and Control (Privacy)
  - TCC Internals
  - CVE-2020-29621 - Full TCC Bypass via coreaudiod
  - Bypass TCC via Spotlight Importer Plugins
  - CVE-2020-24259 - Bypass TCC with Signal to Access Microphone
  - Gain Full Disk Access via Terminal
  - Wrapping Up
10. Symlink and Hardlink Attacks
  - The Filesystem Permission Model
  - Finding Bugs
  - CVE-2020-3855 - macOS DiagnosticMessages File Overwrite Vulnerability
  - CVE-2020-3762 - Adobe Reader macOS Installer Local Privilege Escalation
  - CVE-2019-8802 - macOS Manpages Local Privilege Escalation
  - Wrapping Up
11. Getting Kernel Code Execution
  - KEXT Loading Restrictions
  - Sample KEXT
  - The KEXT Loading Process
  - CVE-2020-9939 - Unsigned KEXT Load Vulnerability
  - CVE-2021-1779 - Unsigned KEXT Load Vulnerability
  - Changes in Big Sur
  - Wrapping Up
12. macOS Penetration Testing
  - Small Step For Man
  - The Jail
  - I am (g)root
  - CVE-2020-26893 - I Like To Move It, Move It
  - Private Documents - We Wants It, We Needs It
  - The Core
  - Wrapping Up
    
