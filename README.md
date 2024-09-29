debugger and several tools for x64/x86 windows.

in local system debugging:
debugger let debug several processes at once, debug or only attach (without actual debugging) to process, for view it memory/modules, let debug protected processes, debug from first instruction in user mode, detach from debugged process, view kernel memory and modules. auto debug child processes, trace function execution, work on winlogon desktop, etc. patially supported source code mode.

supported KDNET network kernel debugging (ipv4 or ipv6). setup guest same as for windbg.
however this mode not very developed and stable - the main mode for me is local debugging

partially supported memory dump analyze (not supported minidump)

The debugger is focused on local system debugging, maximum performance, minimum resource consumption and minimum size.

many things are not intuitive and are not clearly presented in the interface. so using all its functionality will be very problematic, but as it is. the package is designed for a certain debugging style (probably, just as there are different styles of writing code, there can be different styles in this area) and is written for one specific person. in any case, for its effecient use it, a lot of experience and knowledge in the subject is required


debugger:

F11 - step into

F10 - step over
The difference between a call instruction, whether to step inside or not, and a rep

F5 - go

F4 - build a call tree 🌲 - traces all instructions using trace flag until the stack pointer becomes greater than the initial one. Usually it makes sense to press F4 only on the first instruction of a function. The built tree can be saved and viewed later in treeview.exe (it is better to run it as admin the first time, so that it can register the .tvi extension in shell)

F12 - do not process the exception we stopped at, but pass it to application. Usually makes sense for breakpoint, single step exceptions. Some programs, as an anti-debugging trick, deliberately generate such exceptions, because debuggers usually always stop on them and continue with the next instruction. Whereas without a debugger, SEH/VEH would have worked. My debugger allows you to set - do not stop it is a first chance exception for them (i.e. do not process the breakpoint if I did not set it) but if we forgot to do this and understand that this is not a debug assert but specifically against the debugger, then we can press F12.

F9 toggle breakpoint on selected line

In remote debug mode - break/pause button to pause under the debugger.

In the exports/forwards windows ctrl+c copy the list to the clipboard.

From the main dialog we can "load" the kernel driver into the process. Ntoskrnl and other modules. Well, what does load mean.. they are always present in memory anyway. But the debugger does not add them to the process dll list and does not load symbols for them. When you click load - driver ( ntoskrnl first of all) is added to the list of dlls and symbols are loaded, after which it is convenient to look at it

Integration:
If the debugger sees that dll/exe have a link to pdb and it cannot be found (by absolute path, in the same directory as exe and in symbols directory), then a window opens with a suggestion to enter the path to pdb manually. If getpdb.exe is running at this moment - then the link to pdb is automatically copied to its window, and you can start downloading immediately. This is especially convenient when we work on winlogon desktop 🖥️, where copy - paste does not work. If we debug lsass.exe at this moment, then the download will not work ( tls makes calls to lsass), so of course you need to load the symbols in advance. Some other functions, for example, displaying a token will also not work when debugging lsass if it is stopped in the debugger.

In general, when debugging system critical processes, you need to be especially careful, since there is a risk of hang system. If we debug (try to debug) csrss from our session (in which the debugger is running), then we are dependent immediately and forever. Since the debugger itself depends on csrss in its session. But we can debug csrss from 0 session or from another terminal session (there is an RDP connection). Most often, in my practice, you need to debug lsass and there are no problems here. With minor exceptions (show token, open/save dialog), there are also no problems debug winlogon, servicess. You can debug rpcss, dcomlaunch svchost. But here you need to be especially careful. There is a risk of hanging. You need to understand what you are doing. When debugging explorer, before we stop it, you need to press alt-tab 2 times. So that explorer removes its alt-tab handler (modern UI) and switches to classic UI in csrss. If we do not do this, we will have problems switching to another application (alt-tab will not work as well as the task bar, while explorer is stopped). Sometimes it is important to set topmost for the debugger window. In general, there are some subtleties that actually do not relate to the debugger itself, but to the ability to debug in general.

Run as pro.exe allows you to run the process with various options. With another process as a parent, such as low integrity or trusted installer, in another session, on another desktop, with a dll (apc inject, call before exe entry point) and TP. In particular, it allows you to specify my debugger to run the process immediately under the debugger. Which is of course convenient if we need to run several times with the command line (so as not to enter it each time). Or we need to run it in another session, from another process, on another desktop (the debugger itself also supports options when starting the process, but more narrow). In any case, this is useful when testing a program that should be executed as a system, in the 0 session and TP.

SearchEx.exe
Allows you to copy-paste the found results into a special dialog in the debugger and then go to the specified places in the found DLLs. That is, search for strings, binary data, dword, and then go to these places in the DLL. At first glance, this is not a very significant functionality, but in my practice it often turned out to be very useful
