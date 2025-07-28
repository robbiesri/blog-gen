+++
date = '2025-07-26T22:47:17-07:00'
draft = false
title = 'Cpp Reconstruct'
+++

I was refactoring some code to delay init after parsing an external source. I needed to move some of the code out of the constructor, into an init function so I could return a possible failure. *However*, it wasn't trivial to do because some members of the target struct can only be initialized via constructor.

In this case, we have a pattern where we have `public` data members that are constructor-built.

```c++
struct SomeStruct
{
    SomeStruct(int derp)
        : HerpDerp(derp)
    {}

    const int HerpDerp;
};
```

We can avoid writing accessors for these simpler structs, and save ourselves some time when running Debug builds (see [Random ASCII – tech blog of Bruce Dawson](https://randomascii.wordpress.com/2011/07/22/visual-c-debug-buildsfast-checks-cause-5x-slowdowns/)). But I can't move or copy into an already constructed instance.

We talked thru a couple ideas, and settled on using placement new to reconstruct the instance in-place. My co-worker quickly realized we'd be obliged to manually invoke the destructor.

I took a look at StackOverflow, and it had similar ideas as we did. But I did stumble on a beautiful solution from Denis Bider:


```c++
#include <type_traits>

template <class T, typename... Args>
void Reconstruct(T& x, Args&&... args)
{
    static_assert(!std::has_virtual_destructor<T>::value, "Unsafe"); 
    x.~T();
    new (&x) T(std::forward<Args>(args)...);
}
```

The arg forwarding was a nice touch, but the usage of `has_virtual_destructor` was new to me! A version of this is what we ended up using.

I forgot to check with Claude, and it told me something new! `std::construct_at` and `std::destroy_at`. 

Sources:
* [SO: Calling a constructor to re-initialize object](https://stackoverflow.com/a/30460031)
* [`std::construct_at`](https://en.cppreference.com/w/cpp/memory/construct_at.html)
* [`std::destroy_at`](https://en.cppreference.com/w/cpp/memory/destroy_at.html)
