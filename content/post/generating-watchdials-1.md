+++
date = "2017-05-29T17:24:16-07:00"
draft = false
title = "Generating watch dials - Part 1"
tags = [ "svg", "cpp", "watch-building" ]
topics = [ "watch-building", "programming" ]

+++

I've been thinking about building my own watch, after finding that many parts are readily available online, and not crazy expensive. It's partly driven by curiosity, and partly by the fact that I can't find an affordable watch that's looking just the way I want it.

As a first step, I wanted to play with some ideas regarding the dial. I found some services offering laser cutting and PCM etching which I believe I could use, on of them being [Ponoko](http://www.ponoko.com). They are accepting [SVGs](https://www.w3.org/Graphics/SVG/) from Adobe Illustrator and [Inkscape](http://www.inkscape.org). After playing with Inkscape for a bit, I decided to write a simple generator in C++ to programmatically generate some of the elements.


<!--more-->

While a fairly simple hack, I deemed it was interesting enough to push it to github: you can find it [here](https://github.com/winkj/watch-dial-svggen). The code should be pretty straight forward, with an `SvgGen` class which is generating some circles, rects and text elements.

The more interesting part is the actual watch dials. I also included a few sample dials, named `dail_*`. I'd like to briefly walk through one of the samples, and explain it in more detail. Source code here [dial_circletest.h](https://github.com/winkj/watch-dial-svggen/blob/master/dial_circletests.h).

The way I set up the `dial_` functions they're taking a reference to the svg generator (this way, multiple dials can be drawn into the same svg file) as well as offsets for `x` and `y`, so that function itself doesn't have to worry about absolute positioning.

```c++
void dial_circleTests(SvgGen& gen, double originX, double originY)
```

After that, there's a few constants. One thing worth noting is the text attributes:
```c++
std::string textStyle = "text-anchor='middle' alignment-baseline='central' font-size='8'";
```
For one, I implemented a simple string replacement to replace single quotes with double quotes when passing attribute arguments to the svg generator; this way, I don't have to escape all the double quotes. The other thing is the alignment: `text-anchor` is for the horizontal position, `alignment-baseline` for the vertical one.

Afterwards, there's a couple of variables for the various radii, and then a sizes for dot and rectangle decorations.

More interating are the `angleStep` and `angleOffset` variables: the former is calculating the angle between decorations, in radians. A full circle is 2*Pi, so we're dividing this by the 12, the number of segments. The latter is a static offset: since 0 degree in our coordinate system is at 3 o'clock, we'll have to subtract one quarter of the full circle.

```cpp
double angleStep = 2 * M_PI / sections;
double angleOffset = -M_PI / 2; // the coordinate system starts at 3 o'clock
```

Finally, we'll start with the drawing: we'll draw five circles, one as the circle line for the dots, one for the roman numerals, and then a middle line and two offset lines for the rectangles.

```cpp
gen.circle(originX, originY, outerRadius, redCircleNoFill);
gen.circle(originX, originY, innerRadius, redCircleNoFill);

gen.circle(originX, originY, miniRadius+rectOffsetY, redCircleNoFill);
gen.circle(originX, originY, miniRadius,             redCircleNoFill);
gen.circle(originX, originY, miniRadius-rectOffsetY, redCircleNoFill);
```

Now for the fun part: we're looping from 0 to 11 and place our decorations. The dot is pretty straight forward: all we have to do is calculate the `x` and `y` coordinates, which we can do with the [Equation of the Circle](https://en.wikipedia.org/wiki/Circle#Equations). Note that we're calculating `angle` only once per loop cycle.

```cpp
double angle = angleOffset + angleStep * i;

// outer circle: dots
double x = originX + outerRadius * cos(angle);
double y = originY + outerRadius * sin(angle);
gen.circle(x, y, dotRadius);
```   

The second element is the roman numerals. If you're wondering why I'm using "IIII" rather than "IV", it's called the "watchmaker's four", there's an explanation [here](http://www.orientalwatchsite.com/the-watchmakers-four-why-some-watches-use-iiii-instead-of-iv/). The rest is pretty identical like the above, except that we're using `innerRadius` rather than `outerRadius`:

```cpp
// numbers
x = originX + innerRadius * cos(angle);
y = originY + innerRadius * sin(angle);
gen.text(labels[i], x, y, textStyle);
```

Last but not least, the hour/minute marker rectangles. This begins with the familiar point-on-circle calculation, then we drawn an SVG rect. There's a few interesting bits here:

1. We want the rectangle to be rotated, so we use the SVG `transform` function
2. `transform` takes an angle in degrees rather than radians; we're multiplying the number of the segment `i` with 30, because 360 degrees divided by 12 segments it 30.
3. `transform` rotates either around the origin, or a parameter passed to it; we're passing `x` and `y` we got from the equation of the circle
4. SVG's rect's `x` and `y` are it's top-left corner, so we have to move them by the helper variables `rectOffset{X,Y}`, which are just half of the width and height respectively

```cpp
// outer circle: small rectangles, rotated
x = originX + miniRadius * cos(angle);
y = originY + miniRadius * sin(angle);
gen.rect(x-rectOffsetX, y-rectOffsetY, rectWidth, rectHeight,
    "transform='rotate("
    + std::to_string(i * 30)
    + ", " + std::to_string(x)
    + ", " + std::to_string(y)
    + ")'"); // 30 = 360 / 12
```

And this is the end result:

![alt text](/img/test-2.svg)

If your browser can't render SVG, [here](/img/test-2.png) is a PNG version of it.
