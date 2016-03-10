---
layout:     post
date:       2016-03-08 18:00:00
author:     "Jakub Kvita"
comments:   true
title:      "LuaJIT memory limitations"
subtitle:   "How to deal with 2 GB restriction and speed it up even more."
---

Popular fast Lua compiler [LuaJIT](http://luajit.org/) has a design limit which does not allow memory allocator to access more than roughly 2 GB of memory. If it happens LuaJIT stops processing code and fails with following error (or with error even more catastrophic):

~~~ lua
luajit: not enough memory
~~~

This limit is not set in size, but in area of the memory which is available to access. Therefore it is also dependent on how are other processes using resources.

## Garbage collector limitations
Current garbage collector (GC) in LuaJIT 2, which is basically the same as the one in vanilla Lua 5.1 interpreter, is not very fast and it has even worse performance with large workloads. New GC is [already proposed](http://wiki.luajit.org/New-Garbage-Collector) for the LuaJIT 3.0, however that is still long way down the road. Well, what can we do with it now?

Easiest solution would be to switch to basic Lua 5 compiler. Code should be compatible, so no rewriting is necessary, just the change in the underlying machine. Unfortunately, vanilla Lua is not known to be very fast. At least LuaJIT is [much faster](http://luajit.org/performance.html) and that is probably reason, why you are using it. Lets move to the different solution.

## FFI Library

The FFI library allows calling external C functions and using C data structures from pure Lua code ([FFI page](http://luajit.org/ext_ffi.html)). Library is built into LuaJIT and is not shipped as separate module. If you have LuaJIT, you can try `local ffi = require("ffi")`.

Using FFI is quite simple, as you can  just copy and paste C code, which will be compiled. Simple example from the mentioned FFI page, of calling C function:

~~~ lua
-- Load the FFI library. --
local ffi = require("ffi")

-- Add a C declaration for the function. The part inside the double-brackets (in green) is just standard C syntax. --
ffi.cdef[[
int printf(const char *fmt, ...);
]]

-- Call the named C function. --
ffi.C.printf("Hello %s!", "world")
~~~

FFI can be useful in many applications, for example creating and working large C array of integers is much faster and more efficient than using Lua tables. Our problem can be solved by using `malloc` to allocate memory outside of Lua memory management:

~~~ lua
local p = ffi.C.malloc(n)
~~~

See how to use C data structures on the [FFI page](http://luajit.org/ext_ffi.html).

As the memory is unmanaged, we have to free it manually:

~~~ lua
ffi.C.free(p)
~~~

or associate the pointer with a finalizer:

~~~ lua
ffi.gc(p, ffi.C.free)
...
p = nil -- Last reference to p is gone.
-- GC will eventually run finalizer: ffi.C.free(p)
~~~

### ffi.new note

Data structures can be instantiated by calling  `ffi.new`, but this call allocates Lua managed memory and will not bypass the limitation. It is easier and provide access to C data structures, so it is up to you, what you need.

## Torch - notes and solution

[Torch](http://torch.ch/) is by default using LuaJIT, therefore most of the above still apply. However it is necessary to mention that Tensor variables are created by `malloc` and do not count to the memory limit. If you want to switch to Torch using Lua 5, you will need to reinstall - follow instructions [here](https://github.com/torch/distro).

### torch/tds

Torch has a project [_tds_](https://github.com/torch/tds) - Torch C data structures, which is currently using FFI and allocation as discussed above and wraps it to a neat package for using with Torch. Basically new C structures, which correspond to Lua structures (tabkes,...), are created with finalizer already associated. These structures have same interface as any other Lua structure and no need to code in C is required. Torch tensors are also using this package.

First install tds package:

~~~ lua
luarocks install tds
~~~

Then, moving data outside of Lua scope is as easy as:

~~~ lua
tds = require 'tds'
dataInC = tds.Hash(data)

data = nil
collectgarbage()  --clean data variable
~~~

If you load very large data, things will get a little bit more complicated. See the [_torch/tds_](https://github.com/torch/tds) page for more options.
