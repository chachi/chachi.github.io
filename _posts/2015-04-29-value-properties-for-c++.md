---
layout: post
title: Value Properties for C++
---

On Hacker News this morning I came across http://www.nanoant.com/c++/cpp-field-accessors-cure-for-public-field-fobia which tries various ways of improving the hellishly verbose getter/setter situation in C++, but fails to find one that works adequately.

The idea of property functions appeals to my Pythonic leanings - Python allows you to declare methods as `@property` as follows (from the Python 2 [docs](https://docs.python.org/2/library/functions.html)):

```python
class C(object):
    def __init__(self):
        self._x = None

    @property
    def x(self):
        """I'm the 'x' property."""
        return self._x

    @x.setter
    def x(self, value):
        self._x = value

    @x.deleter
    def x(self):
        del self._x
```

Those few lines allow a user to access `myvar.x` as if it were a regular public field, while all accesses transparently flow through these functions. This allows you to do all sorts of things behind the scenes (e.g. caching, dynamic properties, logging) while still presenting a clean public API for your objects.

I think I've found a better way to do this in C++, without too much bastardizing of normal program structures. Using C++'s delegating constructors and in-class initialization of non-static data members, we can do the following (tested with default Clang on MacOS 10.10, using `--std=c++11`):

```c++
#include <iostream>

template <typename T>
struct Property {
  Property(T* t) : parent(t) {}

 protected:
  T* parent;
};

struct Type {
  struct : public Property<Type> {
    using Property::Property;
    float operator=(float value) {
      parent->x *= value;
      parent->y *= value;
	  v = value;
      return v;
    }

    operator float() {
      return v;
    }

    float v = 4.0;
  } value = this;

  int x = 3;
  int y = 4;
};

int main(int argc, char** argv) {
  Type a;
  a.value = 3;

  std::cout << a.x << std::endl;
  std::cout << a.value << std::endl;
}
```

When run, this should output:

```
9
3
```

Breaking this down, we start with a templated base class for our properties that captures a pointer to the parent value for use in method calls.

```c++
template <typename T>
struct Property {
  Property(T* t) : parent(t) {}

 protected:
  T* parent;
};
```

Then we see the declaration of the anonymous struct which inherits from `Property<Type>`.

```c++
  struct : public Property<Type> {
    using Property::Property;
      ...
  } value = this;
```

Here our anonymous struct inherits from `Property<Type>` and also inherits its constructor `using ...;`. This allows us on the last line to initialize in line with `value = this;`, calling the base constructor to initialize the `T* parent`.

It would be nice to get away without explicitly initializing `T* parent` somehow, but I'm not sure what that magic would be.

```c++
    float operator=(float value) {
      parent->x *= value;
      parent->y *= value;
	  v = value;
      return v;
    }

    operator float() {
      return v;
    }
```

Finally, we have the actual `operator`s which are our accessors. We can define as many of the different operators here as we'd like to cover all our use cases: `operator*`, `operator*=`, `operator,`, etc. For now, simply exposing an `operator=` and a `operator float()` we can limit the interactions to just setting and retrieving.

With only these two operators defined, for instance, calling `a.value *= 3` is not possible. If we wanted to expose that, we could define a `float& operator*=` or simply turn `operator float()` into `operator float&()` and let it return a reference to the underlying value.
