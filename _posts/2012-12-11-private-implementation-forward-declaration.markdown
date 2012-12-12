---
layout: post
tags: [ c++ ]
title: Forward Declaration and Private Implementation in C++ 
---

{% capture column_left %}

Well... I catually learned this a long time ago, but nevertheless.

Includes in C/C++ are extremely inefficient. It's not uncommon to find a system where a single change to a header file
file would cause 10 minutes recompilation. I used to suffer from this a lot. I could count the number of times my code
had compiled (or not) in a day by the number of tea/coffee cups consumed plus the number of articles read.

There's been a lot of discussion on C++ modules recently. [These slides by Doug Gregor][dg] explain what's happening 
under the hood, why C++ compilation tends to be so slow, and how C++ modules can help. The bottom line is that compiling
even a single source file might take a while.

{% endcapture %}



{% capture column_right %}

<div class="thumbnail centered-text" markdown="1">

![It's Compiling](http://imgs.xkcd.com/comics/compiling.png)

<p>

[XKCD #303][xkcd]

</p>

</div>

{% endcapture %}

{% include two_columns.html %}

#### What's the big deal?

In a typical coding routine one might make a few changes to a single file and then check if the software still compiles. 
Changing a `.cpp` implementation file won't take much time to recompile. But things get worse if a `.h` header is changed. In 
this case all the implementation files that directly or indirectly include the header in question will require a round of 
recompilation (`three.h` might include `two.h` indirectly including `one.h`). A small change might result in 
recompiling hundreds of files. And most of these files wouldn't use anything from the modified header.  
Time to get another coffee, read an article, or engage in a sword fight with your colleague.

While it's hard to make GCC compile stuff faster, there are a few things that can be done to optimize the compilation,
namely:

+ Avoid including headers using [Forward Declaration][fd]
+ Avoid changing headers using [Private Implementation][pi]

#### Avoid including headers (Forward Declaration) { #fdecl }

Consider the following code:

{% capture column_left %}
<div class="panel">
{% highlight cpp linenos %}
/* one.h */
class One { 
    public:
        int square(int x);
    ... 
};

/* one.cpp */
#include "one.h"
int One::square(int x) { 
    return x * x; 
}
{% endhighlight %}
</div>
{% endcapture %}



{% capture column_right %}
<div class="panel">
{% highlight cpp linenos %}
/* two.h */
#include "one.h"
class Two {
    public:
        Two(One *one);
        int squareUsingOne(int x);
    ...
    private:
        One *m_One;
};

/* two.cpp */
#include "two.h"
Two::Two(One *one) : m_One(one) { }

int Two::squareUsingOne(int x) { 
    return m_One->square(x); 
}
{% endhighlight %}
</div>
{% endcapture %}

{% include two_columns.html %}

In this example changing the header `one.h` would cause recompilation of all the source files that directly or
indirectly include both `one.h` and `two.h`. If `two.h` is an important header and has lots of dependents then all this
sub tree will require a round of recompilation.

But there is no need to include `one.h` from `two.h` because the definition of `class Two` doesn't need to
know anything about the definition of `class One`. It's enough to know that the constructor argument `One *one` 
is a pointer to an instance of a class... some class defined later on. Thus we can forward declare `class One` 
and move `#include one.h` into the the implementation file `two.cpp`:

{% capture column_left %}
<div class="panel">
{% highlight cpp linenos %}
/* two.h */
class One;
class Two {
    public:
        Two(One *one);
        int squareUsingOne(int x);
    ...
    private:
        One *m_One;
};
{% endhighlight %}
</div>
{% endcapture %}

{% capture column_right %}
<div class="panel">
{% highlight cpp linenos %}
/* two.cpp */
#include "two.h"
#include "one.h"
Two::Two(One *one) : m_One(one) { }

int Two::squareUsingOne(int x) { 
    return m_One->square(x); 
}
{% endhighlight %}
</div>
{% endcapture %}

{% include two_columns.html %}

In this setup we avoid recompilation of all the dependents of `two.h`, which may or may not be a lot of CPU cycles. If it's
not a lot of work - then good for you, your code base is probably well decoupled.  
But quite often this simple technique will save you a lot of time.

#### Avoid changing headers (Private Implementation) { #pimpl }

Lets consider a similar but slightly different case:

{% capture column_left %}
<div class="panel">
{% highlight cpp linenos %}
/* one.h */
class One { 
    public:
        int square(int x);
    ... 
};

/* one.cpp */
#include "one.h"
int One::square(int x) { 
    return x * x; 
}
{% endhighlight %}
</div>
{% endcapture %}


{% capture column_right %}
<div class="panel">
{% highlight cpp linenos %}
/* two.h */
#include "one.h"
class Two {
    public:
        int squareUsingOne(int x);
    ...
    private:
        One m_One;
};

/* two.cpp */
#include "two.h"
int Two::squareUsingOne(int x) { 
    return m_One.square(x); 
}
{% endhighlight %}
</div>
{% endcapture %}

{% include two_columns.html %}

In this case `class One` is an instance member of `class Two`. Here the compiler has to know the definition of `class Two` 
in order to include an instance as a member of `class One`. Thus `two.h` has to include `one.h`...

Unless all the private business of `class Two` is hid using the Private Implementation pattern!
Lets reorganize `two.h` and `two.cpp` a little bit:

{% capture column_left %}
<div class="panel">
{% highlight cpp linenos %}
/* two.h */
class TwoPrivate;

class Two {
    public:
        Two();
        ~Two();

        int squareUsingOne(int x);
    ...
    private:
        Two(const Two &two);
        Two &operator=(const Two &two);
        TwoPrivate *d;
};
{% endhighlight %}
</div>
{% endcapture %}


{% capture column_right %}
<div class="panel">
{% highlight cpp linenos %}
/* two.cpp */
#include "two.h"
#include "one.h"
class TwoPrivate {
    public:
        TwoPrivate(Two *owner) 
            : m_Owner(onwer) {}
        ~TwoPrivate() {}

        One m_One;
    private:
        Two *m_Owner;
};

Two::Two() { d = new TwoPrivate(this); }
Two::~Two() { delete d; }

int Two::squareUsingOne(int x) { 
    return d->m_One.square(x); 
}
{% endhighlight %}
</div>
{% endcapture %}

{% include two_columns.html %}

What happened here? A few things.

- `class Two` doesn't have `class One` as an instance member anymore,
- `class Two` now has a pointer member to `class TwoPrivate`. It's [forward declared](#fdecl) because we only deal with a pointer,
- `class Two` has disabled copy constructor and assignment operator in order to [avoid memory leaks and double
  deletion][reddit].
- `class TwoPrivate` is instantiated and destroyed along with `class Two` and is accessible via the `TwoPrivate *d` pointer,
- `class One` is now an instance member of `class TwoPrivate`, accessible in `class Two` members via `d->m_One`,
- Consequently the troubling `#include "one.h"` has been moved from the header `two.h` to the implementation `two.cpp`
  slashing a branch of the inclusion tree.

There are quite a few benefits from using this pattern. To name a few

- The implementation of the class that uses private implementation doesn't affect the interface at all! Members and
  methods can now be added and removed, and only the `.cpp` implementation file will require recompilation,
- The header is now much cleaner - without all the private litter,
- It's easier to maintain binary compatibility. If you work on a dll that's called by third party application, then one
  can safely change the implementation, add/remove private members and methods, and have an interface that's compatible
  with previous versions.

There some drawback as well

- There is a slight increase in memory allocation - from 12 bytes per object depending on the platform.
- Everything that belongs to the Private Implementation will be allocated on the heap. Heap allocation is usually much 
  more expensive than stack allocation. Frequent and long lasting heap allocations often lead to dramatic memory 
  framentation further decreasing performance.
- There is additional pointer indirection every time Private Implementation member or method is accessed.

Private Implementation alone is an extended topic. The solution in this article has many flaws, but it's probably the
simplest implementation possible that gives more or less clear idea of the concept. Proper solution that is at the same time 
correct, consice, and elegant probably just doesn't exist. There is [Loki Pimpl][loki] class that's takes care of all the 
edge cases. It's really a matter of taste and preference. Use the correct but complex code or the simple but flawed 
in some way? Use Private Implementation or not to use it at all?

#### Conclusion

Forward Declaration and Private Implementation can significantly reduce header dependencies within a C++ project, 
save a lot of CPU cycles avoiding unnecessary recompilation, and make your code a lot cleaner.

[dg]: http://llvm.org/devmtg/2012-11/Gregor-Modules.pdf
[xkcd]: http://xkcd.com/303/
[fd]: http://en.wikipedia.org/wiki/Forward_declaration
[pi]: http://en.wikipedia.org/wiki/Private_class_data_pattern
[reddit]: http://www.reddit.com/r/programming/comments/14no5v/forward_declaration_and_private_implementation_in/c7ey70s
[loki]: http://loki-lib.cvs.sourceforge.net/viewvc/loki-lib/loki/include/loki/Pimpl.h?view=markup
