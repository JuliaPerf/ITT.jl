# IntelITT.jl

*Julia interface for the Intel Instrumentation and Tracing APIs*

IntelITT.jl provides a high-level Julia interface to the [Intel Instrumentation and Tracing
Technology APIs](https://github.com/intel/ittapi) APIs. These make it possible to control
and enrich the collection of trace data when profiling Julia code with Intel tools such as
VTune.

When importing IntelITT.jl, a library that provides stub implementations of the IntelITT
APIs will be loaded. In the case your application is being profiled, these stubs will be
replaced by an actual implementation. This means that you should always be able to use
IntelITT.jl without having to worry about whether you are being profiled or not.


## Collection control

When actually using Intel tools to profile your application, it's recommended to start the
profiler in paused mode, and use IntelITT.jl to only instrument the parts of your code that
you are interested in. This can be done as follows:

```julia
using IntelITT

# warm-up, compile code, etc

IntelITT.resume()
# do interesting things here
IntelITT.pause()

# convenience macro for the above:
IntelITT.@collect begin
    # ...
end
```

## Trace instrumentation

IntelITT.jl can also be used to enrich the trace data with custom information. For example,
using tasks you can annotate a logical unit of work (e.g., a function call):

```julia
dom = Domain("MyApplication")
task_begin(dom, "my task")
# ...
task_end(dom)

# or, using a convenience macro:
IntelITT.@task "my task" begin
    # ...
end
```

Events can be used to observe when demarcated events occur in your application, or to
identify how long it takes to execute demarcated regions of code:

```julia
ev = Event("my event")
start(ev)
# ...
stop(ev)
```

If you need to keep track of a value, you can use counters:

```julia
ctr = Counter{Float64}("my counter", "MyApplication")
ctr[] = 0.0
# ...
ctr[] = 1.0

# when using an UInt64 counter, you can increment/decrement it:
ctr = Counter{UInt64}("my counter", "MyApplication")
ctr[] = 0
# ...
increment!(ctr, #=delta=1=#)
decrement!(ctr, #=delta=1=#)
```


## Attaching to launched processes

If you want to attach a profiler to a running process, you need to take extra care, as the
profiler will not be able to override the stubs provided by IntelITT.jl. To work around
this, you need to specify beforehand which ITT API collector to use. For example, if you
know you'll be attaching using VTune, find the collector library that VTune uses:

```
$ INTEL_LIBITTNOTIFY64=~/intel/oneapi/vtune/latest/lib64/runtime/libittnotify_collector.so \
  julia
```


## Running Julia under VTune

In addition to the instrumentation added using IntelITT.jl, it is also possible to have
Julia's JIT compiler emit instrumentation that tools like VTune can use. This requires
starting Julia with the environment variable `ENABLE_JITPROFILING` set to `1`. On older
versions of Julia, pre-1.9, you also need to re-compile Julia from source with the build
option `USE_INTEL_JITEVENTS` set to `1`.

More information, including, e.g., a [Intel VTune remote usage
example](https://juliahpc.github.io/user_hpcprofiling/intel_vtune/) can
be found in the [Julia On HPC Clusters](https://juliahpc.github.io/)
notes.


## Acknowledgments

- https://discourse.julialang.org/t/using-the-intel-vtune-profiler-with-julia/34327/17
- https://github.com/mchristianl/IntelITT.jl

