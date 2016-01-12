
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


# Ruby

https://github.com/ruby/ruby/blob/trunk/array.c

## Growth Pattern

Haven't looked through all of the source, but at looks as if we starts at 16 
(`#define ARY_DEFAULT_SIZE 16`, l 29) and double the size as we go...

`ary_double_capa`, ll 245-258

```c
static void
ary_double_capa(VALUE ary, long min)
{
    long new_capa = ARY_CAPA(ary) / 2;

    if (new_capa < ARY_DEFAULT_SIZE) {
	new_capa = ARY_DEFAULT_SIZE;
    }
    if (new_capa >= ARY_MAX_SIZE - min) {
	new_capa = (ARY_MAX_SIZE - min) / 2;
    }
    new_capa += min;
    ary_resize_capa(ary, new_capa);
}
```

`ary_ensure_room_for_push`, ll 353-

```c
static VALUE
ary_ensure_room_for_push(VALUE ary, long add_len)
{
    long old_len = RARRAY_LEN(ary);
    long new_len = old_len + add_len;
    long capa;

    if (old_len > ARY_MAX_SIZE - add_len) {
	rb_raise(rb_eIndexError, "index %ld too big", new_len);
    }
    if (ARY_SHARED_P(ary)) {
	if (new_len > RARRAY_EMBED_LEN_MAX) {
	    VALUE shared = ARY_SHARED(ary);
	    if (ARY_SHARED_OCCUPIED(shared)) {
		if (RARRAY_CONST_PTR(ary) - RARRAY_CONST_PTR(shared) + new_len <= RARRAY_LEN(shared)) {
		    rb_ary_modify_check(ary);
		    return shared;
		}
		else {
		    /* if array is shared, then it is likely it participate in push/shift pattern */
		    rb_ary_modify(ary);
		    capa = ARY_CAPA(ary);
		    if (new_len > capa - (capa >> 6)) {
			ary_double_capa(ary, new_len);
		    }
		    return ary;
		}
	    }
	}
    }
    rb_ary_modify(ary);
    capa = ARY_CAPA(ary);
    if (new_len > capa) {
	ary_double_capa(ary, new_len);
    }

    return ary;
}
```

ll 1112

```c
capa = ARY_CAPA(ary);
if (capa - (capa >> 6) <= new_len) {
	ary_double_capa(ary, new_len);
}
```

## Interesting Finds

Code is pretty macro heavy
