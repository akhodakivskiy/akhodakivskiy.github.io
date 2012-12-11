---
layout: post
tags: [ c++ ]
title: Forward Declaration and Private Implementation in C++ 
---

{% capture column_left %}

Includes in C/C++ are extremely inefficient. It's uncommon to find a system where a single hange to a header file
file would cause 10 minutes recompilation. I used to suffer from this a lot. I could count the number of times my code
had compiled (or not) in a day by the number of tea/coffee cups consumed plus the number of articles read.

There's been a lot of discussion on C++ modules recently. [These slides by Doug Gregor][dg] explain what's happening 
under the hood, why C++ compilation tends to be so slow, and how C++ modules can help. The bottom line is that compiling
even a single source file take a while.

{% endcapture %}



{% capture column_right %}

<div class="panel" markdown="1">

![It's Compiling](http://imgs.xkcd.com/comics/compiling.png)
[Source: XKCD #303][xkcd]

</div>

{% endcapture %}

{% include two_columns.html %}

#### What's the big deal?

In a typical coding routine one might make a few changes to a sinlge file and then check if the software still compiles. 
Changing a `.cpp` implementation file won't take much time to recompile. But things get worse if a `.h` header is changed. In 
this case all the implementation files that directly or indirectly include the hedaer in question will require a round of 
recompilation (`three.h` might include `two.h` indirectly including `one.h`). A small change might result in 
recompiling hundreds of files. And most of these files wouldn't use anything from the modified header.  
Time to get another coffee, read and article, or engage in a sword fight with your colleague.

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

In this example changing the header `one.h` would cause recomplication of all the source files that directly or
indirectly include both `one.h` and `two.h`. If `two.h` is an important header and has lots of dependents then all this
subtree will require a round of recompilation.

But there is no need to include `one.h` from `two.h` because the definition of `class Two` doesn't need to
know anything anout the definition of `class One`. It's enough to know that the constructor argument `One *one` 
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

In this setup we avoid recompilation fo all the dependents of `two.h`, which may or may not be a lot of CPU cycles. If it's
not a lot of work - then good for you, your codebase is probably well decoupled.  
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
- `class TwoPrivate` is instantiated and descroyed along with `class Two` and is accessible via the `TwoPrivate *d` pointer,
- `class One` is now an instance member of `class TwoPrivate`, accessible in `class Two` members via `d->m_One`,
- Consequently the troubling `#include "one.h"` has been moved from the header `two.h` to the implementation `two.cpp`
  slashing a branch of the inclusion tree.

There are quire a few benefits from this pattern. To name a few,

- The implementation of the class that uses private implementation doesn't affect the interface at all! Members and
  methods can now be added and removed, and only the `.cpp` implementation file will require recompilation,
- The header is now much cleaner - without all the private litter,
- It's easier to maintain binary compatibility. If you work on a dll that's called by third party application, then one
  can safely change the implementation, add/remove private members and methods, and have an interface that's compatible
  with previous versions.

The only drawback from applying the Private Implementation pattern is a slight increase in memory allocation - 
from 12 bytes more per object depending on the platform .

#### Conclusion

Forward Declaration and Private Implementation can significantly reduce header dependencies within a C++ project, 
save a lot of CPU cycles avoiding unnecessary recompilation, and make your code a lot cleaner.

[dg]: http://llvm.org/devmtg/2012-11/Gregor-Modules.pdf
[xkcd]: http://xkcd.com/303/
[fd]: http://en.wikipedia.org/wiki/Forward_declaration
[pi]: http://en.wikipedia.org/wiki/Private_class_data_pattern
