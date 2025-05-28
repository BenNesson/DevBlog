+++
date = '2025-05-21T15:19:17-07:00'
title = 'Animated SVG in Hugo'
tags = [ 'SVG', 'JavaScript' ]
summary = '…in case any other weirdos need to know…'
+++
I mentioned as much in an edit at the end of [the previous post](/posts/trig "Trig"), but getting JavaScript-animated svgs to show up properly via Hugo was a pain in the ass.

Apparently browsers nowadays are (justifiably) wary of running JavaScript from an SVG in an `img` tag.  An `object` tag works, but good luck figuring out if there's an existing way to tell Hugo you want your svg in an `object` tag instead of an `img`.  What're you gonna search the docs for, "object"?  _Good luck_.[^1]

Anyway, for anyone else facing this, here's what I ended up doing:

(And do note that I'm completely new to Hugo and have way more experience with, e.g., **tracking down problems with dynamic initialization of non-local static variables in C++** than with **making website**, so I don't doubt for a second there's better ways to do this.  Look around, this whole thing is clearly pretty early on.  Plus, as mentioned, this was just heinously obnoxious to try to search for.)

I tried a number of different approaches, but in the end settled on a custom shortcode:

In `/layouts/shortcodes/svg_embed.html`:
```
<object data="{{ .Get "src" }}" type="image/svg+xml" alt="{{ .Inner }}"></object>
```

which I could then use like so:
```
{{</* svg_embed src=./trig.svg */>}}
All three animations composited together into one image.
{{</* /svg_embed */>}}
```

It was important to me to make sure I could get alt text in there, because accessibility.  Not sure the `alt` attribute is the best way to do it, but if not, well… hopefully someone will let me know.  And I went with `.Inner` for that because the alt text for some of them was pretty long, and didn't want to think about how I'd pass big, arbitrarily complex chunks of text via a shortocde argument/parameter/whatever.  I'm sure anything too complicated in that `.Inner` block will cause me headaches, but 🤷‍♀️ I wanted to get something in place already.

Of course just now, to escape the shortcodes in that codeblock, I actually had to escape them with `/*` and `*/`, like so:

````
```
{{</*/* svg_embed src=./trig.svg */*/>}}
All three animations composited together into one image.
{{</*/* /svg_embed */*/>}}
```
````

And then to escape the triple backticks just now:
`````
````
```
{{</*/*/* svg_embed src=./trig.svg */*/*/>}}
All three animations composited together into one image.
{{</*/*/* /svg_embed */*/*/>}}
```
````
`````

How I escaped things to get _that_ one to show up correctly will be left as an exercise for the reader.

[^1]: Once upon a time I was writing some kinda crap to simulate mouse movements by directly firing mouse movement events, and found that a lot of arguments to various related API functions were specified in a unit of measure that corresponds to the minimum distance the mouse has to move for the system to register the movement—a unit of measure that is called "a _Mickey_."  I got so mad about how comprehensively unsearchable a term that was that I had to call my dad[^2] and yell at him about it.

[^2]: He's been a programmer longer than he's been a dad, and I often call him to complain about whatever computer nonsense I've been facing lately.  Some folks have to fix their parent's networks all the time; I get to grouse about POSIX thread-exit destructors with mine.
