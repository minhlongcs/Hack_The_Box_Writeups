
Description: Your organization's SOC team intercepted a suspicious binary during a routine threat hunting operation on a Linux server. The file was found in /var/tmp with an unusual name and was attempting to establish outbound connections. Initial analysis suggests this could be a post-exploitation agent. Your task is to perform static analysis on the binary to identify its capabilities, extract indicators of compromise, and understand the threat actor's infrastructure.

- Personal Summary: 
    - New terms, things to learn: binary file ( what it is, how to analyze ) ; post-exploitation agent; static analysis on binary
    - Goals: Identify the binary capabilities, extract indicators of compromise, and threat actor's infrastructure.

## Learning

- Binary file: A [binary](https://www.techtarget.com/whatis/definition/binary) file is a file whose content is in a binary format consisting of a series of sequential [bytes](https://www.techtarget.com/searchstorage/definition/byte), each of which is eight [bits](https://www.techtarget.com/whatis/definition/bit-binary-digit) in length. The content must be interpreted by a [program](https://www.techtarget.com/searchsoftwarequality/definition/program) or a hardware [processor](https://www.techtarget.com/whatis/definition/processor) that understands in advance exactly how that content is formatted and how to read the [data](https://www.techtarget.com/searchdatamanagement/definition/data). Binary files include a wide range of file types, including [executables](https://www.techtarget.com/whatis/definition/executable-file-exe-file), libraries, graphics, databases, archives and many others.
- How to analyze, work a binary: 
    - Static Analyze: 
    - Dynamic Analyze
    - Need to learn: 
        - Assembly language:
        - File format: ELF (Linux), PE(Windows)
        - Operating systems: interact with memories and kernal through syscall
- Basic Binary Analysis tools:
    - file: indentify
    - binwalk: similar tool created to search a given binary for embedded files and executable code. It is compatible with file signatures recognized by the file utility but also comes with a custom signature file for items that are frequently included in firmware images.
    - readelf and ldd: inspect details of a binary's header to find out which shared objects a binary depends on and which library versions the binary expects.
    - xxd: display a given binary
    - nm: list both static and dynamic symbols in a given binary, object file and shred object and demangle symbol names.
    - strings: to look for hints about a file's functionality . Legitimate programs almost always include many strings, like messages, URLs or file locations. Malware that is packed or obfuscated contains very few strings.
    - Specific functions used by an executable provide hints to the functionality of the program, hence inspecting the system and library calls executed by a binary using [**strace**](https://linux.die.net/man/1/strace) and [**ltrace**](https://linux.die.net/man/1/ltrace) can often give you a good high-level idea of what a program is doing.

## Doing the sherlock

"""
┌──(kali㉿kali)-[~/HTB/Sherlocks/PhantomRing/phantom_ring]
└─$ file agent
agent: ELF 64-bit LSB pie executable, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, BuildID[sha1]=1f617f2ea259a7ec724d7bbc01627982dc2f0495, for GNU/Linux 3.2.0, not stripped
""" 

### Question 1: What is the SHA256 hash of the malicious binary?

└─$ sha256sum agent 
2d7b1b2178f76c26893b2a56cbf9b36700235259e76b893d53817d5b66b634a5  agent
