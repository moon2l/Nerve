# Nerve is a simple cross platform (Win32, Linux, OSX) x86 scriptable debugger

## What is it?

    Nerve is based on, and requires, Ragweed http://github.com/tduehr/ragweed
    Ragweed is a cross platform x86 debugging library written in Ruby.

    To learn more about Ragweed, read this:
    http://chargen.matasano.com/chargen/2009/8/27/ruby-for-pentesters-the-dark-side-i-ragweed.html

    Nerve can be a dynamic hit tracer, an in memory fuzzer or a simple scriptable debugger.
    All you need to do is give it a configuration file telling it what breakpoints to
    set, events to hook and what ruby scripts to execute when it happens.

    Nerve showcases the best part about Ragweed: cross platform debugging. I originally
    wrote Nerve as a small Ragweed script that kept stats on the functions my fuzzers
    were triggering in a target process. This told me what code paths my fuzzer was
    reaching and which ones it wasn't. It only took a few hours to make it work on all Ragweed
    supported platforms, and since then it has grown into a much more capable tool. It now
    supports configuration files for breakpoints, event handler scripts and more.

    We have included several working examples with Nerve so that you aren't lost the first
    time you try it. If you develop some useful scripts with it let us know and we can make
    them part of the default package.

## Supported Platforms

    Nerve is supported and has been tested on the following platforms:

    Windows XP, 7
    Ubuntu Linux 9.10 -> 11.04
    Mac OS X 10.5, 10.6

    Ruby 1.8.7
    Ruby 1.9.x

## Features

    - Cross platform
    - Easy configuration files you can write by hand or generate using our tools
    - Run Ruby scripts with full access to the debugger when breakpoints are hit
    - Run Ruby scripts when specific debugging events occur
    - Extend Nerve through handlers.rb or output.rb with minimal code changes
    - Nerve comes with a few example breakpoint scripts such as hooking RtlAllocateHeap/malloc

## Todo / Ideas

	Nerve is a simple tool, but the plan is to grow it with optional add ons and more...

    - Lots of helper scripts for breakpoints such as heap inspection, in memory fuzzing, SSL reads etc...
    - Helper methods and better named instance variables for making breakpoint scripts easier to write
    - Better output such as graphviz, statistics, function arguments etc...
	- Redis database support for offline analysis of output
    - Cleaner 1.8 and 1.9 support for launching processes
    - Continous re-attach to any targets that match the process name
    - Change the configuration to Ruby code (the text file will be an optional fallback)
    - Nerve is also helping us find the areas of Ragweed that need the most improvement

## Requirements

    Nerve has one small dependency. But don't worry, theres no need to install an SQL server
    or compile any code! The dependency, Ragweed, can be installed via Ruby gems on any platform.

    Ragweed (a cross platform x86 debugger library)

    git clone http://github.com/tduehr/ragweed.git    (the preferred method)

    ... OR ...

    gem install -r ragweed   (you might get an older version!)

    Ragweed requires FFI which you can install with rubygems:

    gem install -r ffi

    YES, thats it!

    If you want to run the bleeding edge stuff we commit to github everyday then I suggest
    checking out the github repositories of both Nerve and Ragweed and executing a 'git pull'
    before using the tool. But we can't promise it will work perfectly.

## Usage

    $ ruby nerve.rb  --help

    Nerve 1.9 | Chris Rohlf 2009-2011

    -p, --pid PID/Name               Attach to this pid OR process name (ex: -p 12345 | -p gcalctool | -p notepad.exe)
    -x, --exec_proc FILE             Launch a process according to the configuration found in this file
    -b, --config_file FILE           Read all breakpoints and handler event configurations from this file
    -o, --output FILE                Dump all output to a file (default is STDOUT)
    -f                               Optional flag indicates whether or not to trace forked child processes (Linux only)

## Configuration File Example

    Keywords in configuration files:
    (the order does not matter but each line represents a unique breakpoint)

    bp - An address (or a symbolic name for Win32) where the debugger should set a breakpoint
    name - A name describing the breakpoint, typically a symbol or function name
    lib - An optional library name indicating where the symbol can be found, only useful with Linux/OSX
    bpc - Number of times to let this breakpoint hit before uninstalling it
    code - Location of a script that holds ruby code to be executed when this breakpoint hits
    nargs - The number of arguments the function takes (only used with Win32)
    hook - A true or false configuration to hook both a function entry and exit (Win32 only)

    --

    Win32 Configuration Example:
    bp=0x12345678, name=SomeFunction, bpc=2, code=scripts/SomeFunctionAnalysis.rb, hook=true
    bp=kernel32!CreateFileW, name=CreateFileW, code=scripts/CreateFileW_Analysis.rb

    Linux Configuration Example:
    bp=0x12345678, name=function_name, lib=ncurses.so.5.1, bpc=1, code=scripts/ncurses_trace.rb
    name=malloc, lib=/lib/tls/i686/cmov/libc-2.11.1.so, bpc=10, bp=0x006ff40, code=scripts/malloc_linux.rb

    OS X Configuration Example:
    bp=0x12345678, name=function_name, bpc=6

## Process Launching Configurations

    You can instruct Nerve to launch a target process with arguments and environment
    variables of your choosing. Nerve takes the -x flag along with a filename containing
    your configuration. Be aware that Nerve currently uses exec() to launch processes on
    nix. This means stdout will be written to by the new process. Supporting Process.spawn()
    is easy but its also not Ruby 1.9 compatible. This is on my list to rework!

    Process launching configuration keywords

    target - The location of the application you want to run
    args - A string of arguments to pass to the application
    env - A string of environment variables for the application

    target: /usr/bin/gcalctool
    args: -s 1+1
    env: BLAH=test
    env: MYLIBPATH=/usr/lib

## Breakpoint Scripts

    Nerve supports breakpoint scripts that run when a breakpoint you have specified is executed. These
    can be specified using the 'code=' keyword in your Nerve configuration file (see above).
    These scripts run within the scope of Nerve and the Ragweed breakpoint. This means your scripts
    have access to all the helper methods and instance variables Ragweed makes available. Documenting
    each of these is going to take a bit of time but heres some stuff you can start with.

    Helper Methods:

    (please refer to Ragweed sources for now http://github.com/tduehr/ragweed)

    Instance Variables:

    @ragweed - The Ragweed instance, use this to call all Ragweed methods

    Win32 Specific:
        evt - A debugger event
        ctx - A context structure holding registers
        dir - a string indicating function 'enter' or 'leave'

## Event Handlers Configuration Example

    Event handler scripts work just like breakpoint file scripts. They have full access to the debugger
    but are triggered when specific debug events occur such as 'on_load_dll'. See handlers.rb for how
    they are implemented. Some are OS specific and just wont trigger if you are on a different platform.

    Keywords for configuration files:

    on_access_violation
    on_alignment
    on_attach
    on_bounds
    on_buffer_overrun
    on_breakpoint
    on_continue
    on_create_process
    on_create_thread
    on_detach
    on_divide_by_zero
    on_exit
    on_exit_process
    on_exit_thread
    on_fork_child
    on_guard_page
    on_heap_corruption
    on_illegal_instruction
    on_int_overflow
    on_invalid_disposition
    on_invalid_handle
    on_load_dll
    on_iot_trap
    on_output_debug_string
    on_priv_instruction
    on_rip
    on_segv
    on_sigchild
    on_signal
    on_sigstop
    on_sigterm
    on_sigtrap
    on_single_step
    on_stack_overflow
    on_stop
    on_unload_dll

    This example will run the My_OnLoad_DLL.rb script whenever the LOAD_DLL debug event occurs:

    on_load_dll=scripts/My_OnLoad_DLL.rb

## Examples

    Heres some example output from Nerve running on Ubuntu Linux:

    chris@ubuntu:/# ruby nerve.rb -b example_configuration_files/generic_ubuntu_910_libc_trace.txt -p test
    Nerve ...
    Setting breakpoint: [ 0x0964f40, malloc /lib/tls/i686/cmov/libc-2.11.1.so ]
    Setting breakpoint: [ 0x08055590, mp_add ]
    Setting breakpoint: [ 0x0971830, wmemcpy /lib/tls/i686/cmov/libc-2.11.1.so ]
    Setting breakpoint: [ 0x0969f20, memcpy /lib/tls/i686/cmov/libc-2.11.1.so ]
    Setting breakpoint: [ 0x0964e60, free /lib/tls/i686/cmov/libc-2.11.1.so ]
    Setting breakpoint: [ 0x09b2de0, read /lib/tls/i686/cmov/libc-2.11.1.so ]
    Setting breakpoint: [ 0x09b2e60, write /lib/tls/i686/cmov/libc-2.11.1.so ]
    ^CDumping stats
    0x0a3cf40 - malloc | 5279
    0x08055590 - mp_add | 0
    0x0a49830 - wmemcpy | 0
    0x0a41f20 - memcpy | 0
    0x0a3ce60 - free | 8385
    0x0a8ade0 - read | 0
    0x0a8ae60 - write | 0
    ... Done!

    Here is Nerve running on Windows 7 and debugging an example program that calls HeapAlloc. For
    this test program we want to run a simple ruby script each time we enter and leave RtlAllocateHeap.
    This script should extract the arguments to the function upon entry and the return values on exit.

    ...
    #include <stdio.h>
    #include <windows.h>

    int main(int argc, char *argv[])
    {
       void *a;
       HANDLE h1 = HeapCreate(0, 1024, 1024);
       int i = atol(argv[1]);

       while(1)
       {
           a = HeapAlloc(h1, HEAP_ZERO_MEMORY, i);
           HeapFree(h1, 0, a);
       }

        return 0;
    }
    ...

    Here is the configuration file:

    ...
    bp=ntdll!RtlAllocateHeap, name=RtlAllocateHeap, code=scripts/RtlAllocateHeap.rb, hook=true
    ...

    And here is the scripts/RtlAllocateHeap.rb referenced in the configuration file:

    ...
    ## This script is for Win32 RtlAllocateHeap

    begin
      if dir.to_s =~ /enter/
        @log.str "RtlAllocateHeap -> Size requested #{@ragweed.process.read32(ctx.esp+12)}"
        @log.str "RtlAllocateHeap -> Heap handle is @ #{@ragweed.process.read32(ctx.esp+4).to_s(16)}"
      else
        @log.str "RtlAllocateHeap <- Heap chunk returned @ #{ctx.eax.to_s(16)}"
      end
    rescue =>
      puts "Does your configuration use hook=true?"
    end
    ...

    Below is the output of hooking the malloc.exe program:

    PS C:\Nerve> ruby .\nerve.rb -p test.exe -b .\example_configuration_files\Win32_notepad.txt
    Nerve ...
    Setting breakpoint: [ ntdll!RtlAllocateHeap, RtlAllocateHeap ]
    RtlAllocateHeap -> Size requested 1024
    RtlAllocateHeap -> Heap handle is @ 750000
    RtlAllocateHeap -> Heap chunk returned @ 750590
    RtlAllocateHeap -> Size requested 1024
    RtlAllocateHeap -> Heap handle is @ 750000
    RtlAllocateHeap -> Heap chunk returned @ 750590
    RtlAllocateHeap -> Size requested 1024
    RtlAllocateHeap -> Heap handle is @ 750000
    RtlAllocateHeap -> Heap chunk returned @ 750590         <- ( This is where I CTRL+C the test program )
    RtlAllocateHeap -> Size requested 24
    RtlAllocateHeap -> Heap handle is @ 470000
    RtlAllocateHeap -> Heap chunk returned @ 47f640

    CONTEXT:
    EIP: 77b564f4

    EAX: 000000c0
    EBX: 7ffd3000
    ECX: 77b6350f
    EDX: 00000000
    EDI: 00000000
    ESI: 002af704
    EBP: 002af728
    ESP: 002af6c0
    EFL: 00000000000000000000001000000010 cvvavrxniiodItszxaxpXc
    Dumping stats
    Pid is 3224
    Tid is 4048
    ntdll!RtlAllocateHeap - RtlAllocateHeap | 4

## Useful Tips

    - If you need to declare some global variables you should do it in an on_attach
      script. This code will only run once when the debugger attaches to the target.

    - On windows you can launch Nerve before launching your process. This avoids the
      the need for a process launching script. This is a side effect of how Ragweed
      is designed. This should be the default for all platforms in the future.

## Disassembly

    Ragweed and Nerve do not ship with a disassembly library. We feel that sort of lies outside of the
    scope of a core debugger library. We do however recommend the following Ruby disassembly libraries:

    https://github.com/sophsec/ffi-udis86 - FFI UDis86 library
    http://github.com/struct/frasm - A Ruby C extension for distorm64

## Who

Nerve was written by Chris Rohlf with contributions from AlexRad and Timur Duehr

Ragweed was written by Thomas Ptacek, ported to OSX by Timur Duehr and ported to Linux by Chris Rohlf

Thanks to the www.Matasano.com team and a few other individuals for providing feedback and ideas
