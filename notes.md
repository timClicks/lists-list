
# CPython

https://hg.python.org/cpython/file/tip/Objects/listobject.c

## Growth Pattern

Described on [line 47](https://hg.python.org/cpython/file/tip/Objects/listobject.c#l47):

> The growth pattern is:  0, 4, 8, 16, 25, 35, 46, 58, 72, 88, ...

## Trade Offs / Assumptions

`list_resize` docstring, ll 12-23

>     /* Ensure ob_item has room for at least newsize elements, and set
>     * ob_size to newsize.  If newsize > ob_size on entry, the content
>     * of the new slots at exit is undefined heap trash; it's the caller's
>     * responsibility to overwrite them with sane values.
>
>     * The number of allocated elements may grow, shrink, or stay the same.
>     * Failure is impossible if newsize <= self.allocated on entry, although
>     * that partly relies on an assumption that the system realloc() never
>     * fails when passed a number of bytes <= the number of bytes last
>     * allocated (the C standard doesn't guarantee this, but it's hard to
>     * imagine a realloc implementation where it wouldn't be true).
>
>     * Note that self->ob_item may change, and even if newsize is less
>     * than ob_size on entry.
>     */

## Interesting Finds

Looking at `hg annotate`, we can see that Guido's opening and closing braces 
seem to be one of the few things that remain untouched.
