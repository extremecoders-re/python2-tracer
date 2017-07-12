# python2-tracer

This is a modified version of Python 2.7.13 to support bytecode tracing. 

See this blog post for details: https://0xec.blogspot.com/2017/03/hacking-cpython-virtual-machine-to.html

## Changes from the original source

The only change is in the function `maybe_call_line_trace` in file `ceval.c`

### Original

```c
static int
maybe_call_line_trace(Py_tracefunc func, PyObject *obj,
                      PyFrameObject *frame, int *instr_lb, int *instr_ub,
                      int *instr_prev)
{
    int result = 0;
    int line = frame->f_lineno;

    /* If the last instruction executed isn't in the current
       instruction window, reset the window.
    */
    if (frame->f_lasti < *instr_lb || frame->f_lasti >= *instr_ub) {
        PyAddrPair bounds;
        line = _PyCode_CheckLineNumber(frame->f_code, frame->f_lasti,
                                       &bounds);
        *instr_lb = bounds.ap_lower;
        *instr_ub = bounds.ap_upper;
    }
    /* If the last instruction falls at the start of a line or if
       it represents a jump backwards, update the frame's line
       number and call the trace function. */
    if (frame->f_lasti == *instr_lb || frame->f_lasti < *instr_prev) {
        frame->f_lineno = line;
        result = call_trace(func, obj, frame, PyTrace_LINE, Py_None);
    }
    *instr_prev = frame->f_lasti;
    return result;
}
```
### Changed

```c
static int
maybe_call_line_trace(Py_tracefunc func, PyObject *obj,
                      PyFrameObject *frame, int *instr_lb, int *instr_ub,
                      int *instr_prev)
{
    int result = 0;
    result = call_trace(func, obj, frame, PyTrace_LINE, Py_None);
    *instr_prev = frame->f_lasti;
    return result;
}
```

## Precompiled Version

A precompiled version of the Python dll may be downloaded from the Releases section.

https://github.com/extremecoders-re/python2-tracer/releases/tag/1.0

Copy the dll to `C:\Windows\System32`. 
Make sure to backup the existing dll (if any). When you are done replace with the original dll.
