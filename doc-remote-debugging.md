---
layout: default
title: Remote Debugging
---

ZeroBrane Studio supports **remote debugging** that allows to debug arbitrary Lua applications.
The application may be running on the same or a different computer from the one running a ZeroBrane Studio instance
(the debugger is using a socket interface to interact with the application).

## Remote debugging

* Open ZeroBrane Studio. 
Go to `Project | Start Debugger Server` and **start the debugger server** (if this menu item is disabled, the server is already started).
* **Open the Lua file** you want to debug.
* **Select the project directory** by going to `Project | Project Directory | Choose...`
or using `Project | Project Directory | Set From Current File`.
* Add `require('mobdebug').start()` call to your file.
If the application is running on a **different computer**, you need to specify an address of the computer where ZeroBrane Studio is running as the first parameter to the `start()` call: `require('mobdebug').start("12.345.67.89")` or `require('mobdebug').start("domain.name")`.
You can see the **domain name** to connect to in the Output window when you start debugger server: `Debugger server started at <domain>:8172.`
* Make `mobdebug.lua` and `luasocket` available to your application. This can be done in one of three ways:
(1) Set `LUA_PATH` and `LUA_CPATH` before starting your application (see [Setup environment for debugging](#setup-environment-for-debugging));
(2) Reference path to `mobdebug.lua` and `luasocket` using `package.path` and `package.cpath` (see [Configure path for debugging](#configure-path-for-debugging)); or
(3) Include `mobdebug.lua` with your application by copying it from `lualibs/mobdebug/mobdebug.lua` (this assumes your application already provides `luasocket` support).
* **Run your application**. You should see a green arrow pointing to the next statement after the `start()` call in ZeroBrane Studio and should be able to step through the code.

## Setup environment for debugging

You can use a simple script to set `LUA_PATH` and `LUA_CPATH` environmental variables to reference `mobdebug` and `luasocket` files that come with ZeroBrane Studio:

    set ZBS=C:/path/to/ZeroBraneStudio
    set LUA_PATH=./?.lua;%ZBS%/lualibs/?/?.lua;%ZBS%/lualibs/?.lua
    set LUA_CPATH=%ZBS%/bin/windows/x86/?.dll;%ZBS%/bin/windows/x86/clibs/?.dll
    -- set LUA_CPATH=%ZBS%/bin/windows/x86/?.dll;%ZBS%/bin/windows/x64/clibs/?.dll
    myapplication

If you are running this on Linux, make sure you use the same path separator (`;`)
and reference proper location depending on your Linux architecture (replace `x86` with `x64` if you are running this on a 64bit system):

    export ZBS=/opt/zbstudio
    export LUA_PATH="./?.lua;$ZBS/lualibs/?/?.lua;$ZBS/lualibs/?.lua"
    export LUA_CPATH="$ZBS/bin/linux/x86/?.dll;$ZBS/bin/linux/x86/clibs/?.dll"
    ./myapplication

## Configure path for debugging

In a similar way, instead of specifying `LUA_PATH` and `LUA_CPATH`, you can set `package.path` and `package.cpath` (if needed) directly from your script:

    local zbs = "C:/path/to/ZeroBraneStudio"
    -- local zbs = "/opt/zbstudio"
    package.path = package.path .. ";./?.lua;" .. zbs .. "/lualibs/?/?.lua;" .. zbs .. "/lualibs/?.lua"
    package.cpath = package.cpath .. ";" .. zbs .. "/bin/linux/x86/?.so;" .. zbs .."/bin/linux/x86/clibs/?.so"
    -- package.cpath = package.cpath .. ";" .. zbs .. "/bin/windows/x86/?.dll;" .. zbs .."/bin/windows/x86/clibs/?.dll"

## Examples

See [Debugging Wireshark lua scripts](http://notebook.kulchenko.com/zerobrane/debugging-wireshark-lua-scripts-with-zerobrane-studio) for detailed description on how this remote debugging works with Wireshark scripts.

## Troubleshooting

* **How do I find a path to `mobdebug.lua`?**
`mobdebug.lua` is located in `lualibs/mobdebug` directory under your ZeroBrane Studio installation directory.
The location of ZeroBrane Studio is system dependent;
on **Windows** it is the location of the directory you installed ZeroBrane Studio to;
on **Linux** it is `/opt/zbstudio`;
and on **Mac OS X** it is `/Applications/ZeroBraneStudio.app/Contents/ZeroBraneStudio`
(You may need to right click on the application and select `Show Package Contents` to navigate to that location on OS X).

* **I can't step into functions defined in other files in my project.**
You either need to open them in the IDE before you want to step through them, or to [configure](doc-configuration) the IDE to auto-open files requested during debugging using `editor.autoactivate = true`.

* **The host name is detected incorrectly.**
In some rare cases the domain name detected by ZeroBrane Studio cannot be resolved, which prevents the debugger from working.
You can specify the domain name or address you want to use by [configuring](doc-configuration) the IDE with `debugger.hostname="domain"`.

* **I get `dynamic libraries not enabled` error.**
You may get the following error when loading `socket.core` on Linux:
_error loading module 'socket.core' from file '/opt/zstudio/bin/linux/x86/clibs/socket/core.so': dynamic libraries not enabled; check your Lua installation_.
This most likely means that the Lua interpreter you are using was built without `LUA_USE_DLOPEN` option enabled.
You can either enable it or statically link your application with luasocket.

* (Note: you should not see this error if you are using v0.38 or later) **I get "Debugger error: unexpected response after EXEC/LOAD '201 Started ...'".**
This is caused by not having a filename associated with a dynamic chunk loaded by your application.
If you are using `loadstring()`, you should pass a second parameter that is a filename for the fragment (and that file can then be debugged in ZeroBrane Studio if it's placed in the project directory).
If you are using `luaL_loadstring` (which has no option to label the chunk with its file path), you can switch to using `luaL_loadbuffer` to pass that information.
