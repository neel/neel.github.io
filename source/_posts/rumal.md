title: 
  Rumal a header only HTML/CSS/Javascript Generator C++ library
tags:
  - C++
  - XML
  - HTML
  - CSS
  - Javascript
categories:
  - Announcement
date: 2019-12-26 22:45:26
---
Rumal is a C++ library that can generate HTML/CSS/Javascript code from significantly identical C++ syntax. 
Currently it uses `std::string` which is supposed to be replaced with compile time strings. Injecting placeholders, is also planned but not yet implemented.
This will make it usable as a template engine.


```c++
std::cout << 
    div(_id(42) / _class("test"),
        span(_id(43) / _class("test"), "Hello"),
        span("World")
    );
```
<!--more-->

The above code prints 

```html
<div id="42" class="test">
    <span id="43" class="test"> Hello </span>
    World
</div>
```

Rumal can be used to generate CSS too.

```c++
select(".main", 
      display("block") 
    / position("relative"), 
    select(".heading", 
          display("block") 
        / position("relative")
    )
) / select(".container", 
      display("block") 
    / position("relative")
);
```

With the above C++ code the following CSS is generated

```css
.container{
    position: relative; 
    display: block;
}

.main{
    position: relative;
    display: block;
}

.main > .heading{
    position: relative;
    display: block;
}
```

Rumal can also be used as a kind of Quine (self replicating program) that generates Javascript from a significantly similar C++ syntax.

> However the Javascript generation part is incomplete and buggy. I am not getting enough time to complete / fix it. Use at your own risk

```c++
assignable<Node> document("document");
assignable<Node> y("y");
auto script = jQuery(document).ready(function()[
        jQuery(".hallo").click(function()[
            jQuery(y).hide(),
            jQuery(".hello").hide()
        ])
    ]);
```

The above generates a similar javascript code.

```javascript
jQuery(document).ready(function()[
    jQuery(".hallo").click(function(){
        jQuery(y).hide(),
        jQuery(".hello").hide()
    })
});
```

In order to use rumal js first the OO skeleton of the Javascript types has to be specified

```c++
using namespace rumal::js;

struct fn_: callable_<fn_>{
    fn_(): callable_<fn_>("fn"){}
};

struct m1_: callable_<m1_>{
    m1_(): callable_<m1_>("m1"){}
};

struct m2_: callable_<m2_>{
    m2_(): callable_<m2_>("m2"){}
};

struct m4_: callable_<m4_>{
    m4_(): callable_<m4_>("m4"){}
};

struct m5_: callable_<m5_>{
    m5_(): callable_<m5_>("m5"){}
};
```
Five void returning javascript functions `fn, m1, m2, m4, m5` are specified.

```c++
template <typename T>
struct object4_{
    method_<m5_, T> m5;
    
    object4_(const T& pkt): m5(pkt){}
};

template <typename T>
struct object3_{
    method_<m5_, T> m5;
    property_<object4_, T> o4;
    
    object3_(const T& pkt): m5(pkt), o4("o4", pkt){}
};

template <typename T>
struct object2_: iterable_<T, object3_>{
    method_<m4_, T> m4;
    
    object2_(const T& pkt): iterable_<T, object3_>(pkt), m4(pkt){}
};

struct m3_: callable_<m3_, object2_>{
    m3_(): callable_<m3_, object2_>("m3"){}
};

template <typename T>
struct object1_{
    rumal::js::method_<m1_, T> m1;
    rumal::js::method_<m2_, T> m2;
    rumal::js::method_<m3_, T> m3;
    
    object1_(const T& pkt): m1(pkt), m2(pkt), m3(pkt){}
};

struct m0_: rumal::js::callable_<m0_, object1_>{
    m0_(): rumal::js::callable_<m0_, object1_>("m0"){}
};
```

Javascript Classes `object4`, `object3`, `object2`, and `object1` are specified. The `object4` has a method `m5` in it. So any function returning `object4` will have a `.m5()` method callable. Similarly `object3` describes the method `m5` and a property `o4` or type `object4`. So any function `f` returning an instance of `object4` will have `.m5()` as well as `.o4.m5()` accessible.

Finally the `object1` Class have three methods `m1`, `m2` and `m3`.

Following are somw example usages.

```c++
fn_ fn;
m0_ m0;

std::cout << fn(1, 2.5) << std::endl;
std::cout << m0(1, 4.5) << std::endl;
std::cout << m0(1, 4.5).m1(4, 2.7) << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7) << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7).m4 << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7).m4(4, 75.5) << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7, "hello")[1].m5 << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7, "hi")[1].m5("Hallo") << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7, "hi")[1].o4 << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5 << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5(42) << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5() << std::endl;
std::cout << m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5(42) - m0(1, 4.5).m3(4, 2.7, "hi")[1].o4 + fn * m0 << std::endl;
std::cout << (m0(1, 4.5) , m0(1, 4.5).m1(4, 2.7) , m0(1, 4.5).m3(4, 2.7, "hi")[1].o4) << std::endl;
```

The above code generates the following javascript, which is much like printing itself.

```js
fn(1, 2.5)
m0(1, 4.5)
m0(1, 4.5).m1(4, 2.7)
m0(1, 4.5).m3(4, 2.7)
m0(1, 4.5).m3(4, 2.7).m4
m0(1, 4.5).m3(4, 2.7).m4(4, 75.5)
m0(1, 4.5).m3(4, 2.7, "hello")[1].m5
m0(1, 4.5).m3(4, 2.7, "hi")[1].m5("Hallo")
m0(1, 4.5).m3(4, 2.7, "hi")[1].o4
m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5
m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5(42)
m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5()
m0(1, 4.5).m3(4, 2.7, "hi")[1].o4.m5(42) - m0(1, 4.5).m3(4, 2.7, "hi")[1].o4 + fn * m0
(m0(1, 4.5) , m0(1, 4.5).m1(4, 2.7) , m0(1, 4.5).m3(4, 2.7, "hi")[1].o4)
```
In the example below a javascript variable (assignable) `x` is created. The functions `_const` and `_let` generates the `const` and `let` javascript keywords. `<<==`is used as assignment operator instead.

```c++
auto x = rumal::js::assignable<object2_>("x");
std::cout << x << std::endl;
std::cout << _const(x) << std::endl;
std::cout << (_let(x) <<= m0(1, 4.5)) << std::endl;
```
The above C++ code generates the following Javascript
```js
x
const x
let x = m0(1, 4.5)
```

Next is the example of generating the conditional structure in Javascript `[]` in C++ generates `{}` in Javascript

```c++
std::cout << (
        24 + m0(1, 4.5).m3(4, 2.7).m4 + 42 + "Hallo World",
        _if(x >= 1 && 2*x >= 1)[
            m0(1, 4.5), 
            m0(1, 4.5).m1(4, 2.7),
            m0(1, 4.5).m3(4, 2.7, "hi")[1].o4
        ],
        _else(x < 0.5)[
            m0(1, 4.5), 
            m0(1, 4.5).m1(4, 2.7),
            m0(1, 4.5).m3(4, 2.7, "hi")[1].o4
        ],
        _else()[
            m0(1, 4.5), 
            m0(1, 4.5).m1(4, 2.7),
            m0(1, 4.5).m3(4, 2.7, "hi")[1].o4
        ]
    ) << std::endl;
```

[Gitlab](https://gitlab.com/neel.basu/rumal)