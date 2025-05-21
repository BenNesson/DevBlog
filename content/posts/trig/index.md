+++
date = '2025-05-20T16:36:57-07:00'
title = 'Trig'
+++
Once upon a time, a friend of mine shared a gif on Facebook that looked a little something like this:
{{< svg_embed src=./trig-sin.svg >}}
Geometric animation: Cartesian plane from x = [0,8π] and y = [-3.5,3.5], with a unit circle on the left, centered at (-1, 0).  A line passes through the center of the circle and rotates counter clockwise.  There is a dot where the line initially crosses the circle on the right side, and a green line segment stretches horizontally from that dot to a green dot at (x, sin(x)), moving right as the animation progresses.  As this green dot moves, it traces out a green sine wave path behind it.  Overall, the image visually demonstrates that a sine wave is just the vertical component of the point rotating around the circle.
{{< /svg_embed >}}

Her comment was something along the lines of "Someone should make something like this for the other ones."

I had previously messed around with SVG images for another project[^1], and figured it'd be a good technology to use for this kind of geometric image creation, but I didn't have any experience with animating them.  So I started looking around and it turns out you can just embed javascript in SVG and manipulate the DOM directly.

So I started banging things together and made the image above (for sake of completeness) along with these two:

{{< svg_embed src=./trig-tan.svg >}}
Geometric animation: same setup as above, but instead of a green dot where the line crosses the circle, there's a red dot where it crosses the y-axis.  The resulting path traces out tan(x) in red.
{{< /svg_embed >}}

{{< svg_embed src=./trig-cos.svg >}}
Geometric animation: same setup as the sine animation above, but there's a dashed diagonal line through the circle, with a dashed vertical line segment to the diagonal, a small blue dot where they meet, and a horizontal line segment over to (x, cos(x)).
{{< /svg_embed >}}
(This one was hard to represent in an equivalent way.  I'm still not super happy with the diagonal reflection thing, to be honest.)

And all together:

{{< svg_embed src=./trig.svg >}}
All three animations composited together into one image.
{{< /svg_embed >}}

And that's how I got started with JavaScript.

(And this is how I got started embedding images with Hugo.)

((EDIT: **_WOWSERS BOWSERS_** did embedding javascript-animated SVG images turn out to be way more obnoxious than I was anticipating!  I'm glad I did though, because I've been thinking for a while now about doing a series of posts explaining fundamentals of how computers work, for the lay person, and anticipate that being pretty animated-svg-heavy.))

[^1]: It was around the time DC Comics was doing their "Blackest Night" event, where we learned that there were _more_ lantern corps, with different colors and associated with different emotions!  Like the Red Lantern Corps, who were very angry (first thought), or the Orange Lantern Corps, who were… greedy? (wait, is it sins now?) and was maybe just one guy, and of course the emotion associated with the good ol' Green Lantern Corps was "willpower" which isn't even really an emotion anyway…

    At any rate, the whole thing was just kinda dumb, so in protest/to make fun of the whole thing, I invented my own corps, the Brown Lantern Corps, whose associated emotion was "relief" and I learned enough SVG to design a stylistically matching logo (with stink lines and everything)[^2].

    (Sometimes I think about how much of the stuff I ended up working on over the years ultimately, thanks to SVG acting as my gateway to JavaScript, came out of me wanting to make a poop joke to make fun of Dan DiDio.)

[^2]: Sadly, lost to time.