+++
date = '2025-05-20T16:36:57-07:00'
title = 'Trig'
+++
Once upon a time, a friend of mine shared a gif on Facebook that looked a little something like this:
![Geometric animation: Cartesian plane from x = \[0,8π\] and y = \[-3.5,3.5\], with a unit circle on the left, centered at (-1, 0).  A line passes through the center of the circle and rotates counter clockwise.  There is a dot where the line initially crosses the circle on the right side, and a green line segment stretches horizontally from that dot to a green dot at (x, sin(x)), moving right as the animation progresses.  As this green dot moves, it traces out a green sine wave path behind it.  Overall, the image visually demonstrates that a sine wave is just the vertical component of the point rotating around the circle.](/posts/trig/trig-sin.svg)

Her comment was something along the lines of "Someone should make something like this for the other ones."

I had previously messed around with SVG images for another project, and figured it'd be a good technology to use for this kind of geometric image creation, but I didn't have any experience with animating them.  So I started looking around and it turns out you can just embed javascript in SVG and manipulate the DOM directly.

So I started banging things together and made the image above (for sake of completeness) along with these two:

![Geometric animation: same setup as above, but instead of a green dot where the line crosses the circle, there's a red dot where it crosses the y-axis.  The resulting path traces out tan(x) in red.](/posts/trig/trig-tan.svg)

![Geometric animation: same setup as the sine animation above, but there's a dashed diagonal line through the circle, with a dashed vertical line segment to the diagonal, a small blue dot where they meet, and a horizontal line segment over to (x, cos(x)).](/posts/trig/trig-cos.svg)
(This one was hard to represent in an equivalent way.  I'm still not super happy with the diagonal reflection thing, to be honest.)

And all together:

![All three animations composited together into one image.](/posts/trig/trig.svg)

And that's how I got started with JavaScript.

(And this is how I got started embedding images with Hugo.)