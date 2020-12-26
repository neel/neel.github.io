---
title: Rumal (রুমাল) is a header only XML/HTML/SVG/CSS/Javascript Generator C++ library
date: 2019-12-26 22:45:26
tags: 
    - C++
categories:
    - Announcement
---

Rumal is a C++ library that can generate HTML/CSS/Javascript code from significantly identical C++ syntax. 
Currently it uses `std::string` which is supposed to be replaced with compile time strings. Injecting placeholders, is also planned but not yet implemented.
This will make it usable as a template engine.


```
std::cout << 
    div(_id(42) / _class("test"),
        span(_id(43) / _class("test"), "Hello"),
        span("World")
    );
```
<!--more-->

The above code prints 

```
<div id="42" class="test">
    <span id="43" class="test"> Hello </span>
    World
</div>
```

Rumal can be used to generate CSS too.

```
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

```
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

```
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

```
jQuery(document).ready(function()[
        jQuery(".hallo").click(function(){
            jQuery(y).hide(),
            jQuery(".hello").hide()
        })
    ]);
```