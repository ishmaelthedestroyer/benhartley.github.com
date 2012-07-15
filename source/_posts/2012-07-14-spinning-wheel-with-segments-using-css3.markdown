---
layout: post
title: "Spinning wheel with segments using Stylus and Nib (CSS3)"
date: 2012-07-14 12:16
comments: true
categories: [stylus, css3, nib]
---

Use CSS3 transitions and a clever CSS hack to create a spinning wheel made up of individual segments<!-- more -->

## Demo ([view source on GitHub](https://github.com/benhartley/wheel-demo))
Quick and dirty iframe:

<iframe style="width:728px; height:470px;" src="http://d2uqigvzon6qhz.cloudfront.net/"></iframe>

## Intro
I've been working on a fun project for Monotype this week to demonstrate their [Web Fonts](http://webfonts.fonts.com) service, using CSS3 to create a spinning wheel. It's not quite finished, so excuse any glaring bugs, but I thought I'd share how it's made. The whole project only uses a handful of images, the segments of the wheel are individual divs, the text is all live-text. I could have used even fewer images if I'd had more time on it, but I digress... This write-up glosses-over a lot of the detail, so if you really want to know how it works, you're best off looking through the source code, however, hopefully you'll find both helpful.

## Power of Stylus
This project was made infinitely easier by using [Stylus](http://learnboost.github.com/stylus/) and [Nib](http://visionmedia.github.com/nib/) to create the CSS. Stylus is a simple language that compiles to CSS (similar to SASS or Less), and Nib is a library for Stylus that enables you to write way less code. You can view the whole `.styl` file in the [GitHub repo](https://github.com/benhartley/wheel-demo), and I'll be using simplified excerpts here too. Unfortunately, Pygments doesn't seem to have Stylus syntax highlighting, so I've used Ruby, thus the code snippets may have some weird highlighting...

At the top of `/src/styl/style.styl`, there are two lines:

``` ruby Stylus setup
@import nib
global-reset()
```

The first one means Nib will be used when compiling our CSS - more on that later. The second means Eric Meyer's global browser reset CSS rules will be automatically added to our compiled CSS file. How easy is that?

## The Triangles
On to the CSS hack I mentioned earlier. This came to my attention via [Jon Rohan](http://jonrohan.me/guide/css/creating-triangles-in-css/), and looks like it was created originally by, him again, Eric Meyer. For our purposes, the triangles are created as follows:

``` ruby Triangular divs
.triangle
    border-style solid
    border-width 0 0 250px 800px
    border-color transparent
    width 0
    height 0
```

All we then need to do is add a colour to the bottom-border and we have a triangle (this example has been scaled down):

<div style="border-style:solid;border-width:0 0 125px 400px;border-color:transparent;border-bottom-color:#291f33;width:0;height:0;margin-bottom:30px;"></div>

Now, all we need to do is create several of these triangles and rotate them and we have...

## The Wheel
The `#wheel` in my project is made up of 20 `.segment` divs, but you can use as many as you like. Rather than having to calculate the rotation for each segment, we can use a function in Stylus to do the job for us. As there are 20 segments (numbered 0 to 19), the rotation for each segment will be its number multiplied by 18 (18 = 360º divided by 20 segments). 

``` ruby Rotating the segments
num_segs = 20
rotation = 360 / num_segs
rotate_seg(segment)
    segment * rotation
```

Now we have the `rotate_seg` function, we can loop through our segment numbers, pass them to the function and it will return how much we need to rotate that segment to make the wheel. Note 1: Nib enables us to just use `transform` - no vendor prefixes in sight! Note 2: the line beginning `#seg` is not a comment - the syntax highlighter thinks it is though...

``` ruby Loop through segments and apply rotation
for num in 0..19
    #seg_{num}
        transform rotate(rotate_seg(num)) 
```

If we now set `transform-origin` to `bottom left`, then our segments will fan out into a circle.

While we're looping through the segments, we can also use a function to apply a colour to each one of our triangular divs. In my project, the colours were part of the design, so I used a hash that mapped a colour to a segment number. However, with Stylus you could also apply one of the many colour functions to create a cool effect.

``` ruby Apply a colour to each segment
colors = (0 #fcf0e0) (1 #fc6d58) (2 #3f9149) (3 #5cd0e0) (4 #fcd925) (5 #03b845) (6 #8f0f80) (7 #fc35aa) (8 #b5f032) (9 #fc6d58) (10 #1cd9a6) (11 #def01f) (12 #fca932) (13 #3f659a) (14 #e2d62c) (15 #ea1c53) (16 #5cd0e0) (17 #fca932) (18 #1cd9a6) (19 #c3d49e)

segment_color(segment)
    return color[1] if color[0] == segment for color in colors 

for num in 0..19
    .triangle
        border-bottom-color segment_color(num)
```

## Make it Spin
So, now we have a wheel made up of individual segments. But surely the coolest thing is that we can make it spin, and add a custom easing effect to make the spin feel more natural. Given the odd construction of our wheel, it took a bit of fiddling around to find the centre-point for the rotation. You may want to tweak these numbers if you change any features o the wheel. Again, `#wheel` here is not a comment but a CSS declaration.

``` ruby Finding the centre + setting rotation speed
#wheel
    transform-origin 0% 17%
    transition all 5s cubic-bezier(.16,.63,.4,.99)
```

Notice the cubic-bezier curve defined in the easing function of the transition. Read from left to right, the transition declaration says: for all changes to the `#wheel` div, make them last 5 seconds, and use the following curve for easing. The curve starts off almost vertical, then slows sharply to create the effect of a wheel spinning, then gradually slowing down. I use cubic-bezier.com to create curves for easing functions, and you can [view this particular curve here](http://cubic-bezier.com/#.16,.63,.4,.99).

Now, when we change the rotation of the wheel, it will spin for five seconds, and then gracefully slow down to the desired position. We change the rotation by adding one of several classes to `#wheel` and we generate those classes using Stylus:

``` ruby Rotation classes for the wheel
rotate_wheel(segment)
    if even(segment)
        rotate_seg(segment) - 181deg
    else
        rotate_seg(segment) + 179deg

for num in 0..19
    &.show_{num}
        transform rotate(rotate_wheel(20 - num))
```

So, to show segment 9 for example, we just have to add `.show_9` to the `#wheel` div. The `rotate_wheel` function alternates between clockwise and anti-clockwise rotations for even and odd segments to avoid less interesting transitions (e.g. spinning from segment 4 to 5 includes a whole spin, rather than moving a single place).

## Summary
Stylus and Nib really came into their own for me during this project. It would have been fairly boring and laborious creating the wheel using regular CSS, but as you can see above, it didn't take much code at all.
