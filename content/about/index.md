+++
title = 'About the Author'
+++
I'm Ben Nesson, this is my dev blog.  All opinions are my own.  No warrantee expressed or implied, etc.

# Career
## Misc
I went to SDSU[^1] and after a couple summers delivering pizzas[^2] and running a weedeater in a cemetery[^3], I ended up spending three years or so working in a giant video screen factory (regular-sized factory for giant video screens).[^4]

Once I graduated, I moved to Seattle and got a programmer gig at a small company that got bought out by a competitor like twenty minutes after I got there.  Seriously, the sale was a done deal when I was interviewing, but apparently they had to play dumb to not tip their hand to… someone, Iunno.

After those jerks canned me, I worked a year-long contract wasting time running tests in Windows Security.  The commute was a nightmare, so when the contract ended, I swore I'd never go back to Microsoft.

## Microsoft
(Yeah, you saw that coming.)

Microsoft offered me my first job actually paying decent money[^5], as part of the Parallel Computing Platform team, working on the Concurrency Visualizer.  That team was awesome, the product was awesome, and I still miss the halcyon days of the late aughts.  Anything was possible!  Zunes!  Windows Phone!

Then they decided no one cared about parallelism anymore, so after a reorg, I ended up on the VC++ Libraries team, rehabilitating ancient test automation.  I took a pile of twenty-year-old perl scripts that could barely run a cross-platform XP-targeting test pass and juiced that stuff up to be able to target Phone, XBox, whatever you want.  If you've ever wondered if anyone's ever had to figure out how to run batch scripts on a Windows 8 Phone, well \*points at self with thumbs\*.  I once worked 112 hours over 11 straight days to make sure XP targeting was fixed, so if you were writing software targeting Windows XP in 2012 or so, _you're welcome_.

Another re-org, because Microsoft, and I ended up on the VC++ Language Services team, mostly dealing with MSBuild collateral.  So if you've ever wondered if anyone's ever built a code coverage collection tool for MSBuild project files that collects Property/ItemDefinition/Item coverage via MSBuild static analysis API, Target coverage via MSBuild logging API, and Task coverage by _running the .exe the logger is defined in as a separate process that then **attaches to MSBuild as a debugger using the undocumented reflection/emit debugging API that MSBuild used to expose**, setting breakpoints on each Task, and marking them covered as the breakpoints are hit_, well \*points at self with thumbs\*

Microsoft stopped being a good fit for a number of reasons, though, so…

## Tableau
I worked for Tableau[^6] for almost ten years, and was on the same team the entire time, despite several name changes.

We started out as Test Automation Accelerator; then (since we'd spent too much time mucking around in the presentation layer) we were Desktop Client Infrastructure; then we were Archer, and split into multiple sub-teams.  Other than TAA's relatively restricted purview, most of that time we were the "whatever falls through the cracks" team, owning wide swaths of core foundational infrastructure.  A buncha weirdos working on weird stuff, it was the most fun I've ever had.

More significantly, Archer provided me the opportunity to do something I never thought I'd get to do.  Thanks to an extremely supportive manager[^7] I was able to carve out a new role.  We called it Dr. Nesson.

See, I'm _implausibly_ good at figuring out bugs.  It honestly feels like bad writing sometimes.  Bugs in code I know nothing about, features I know nothing about, operating systems and languages I know nothing about.  I'll figure out an intractable bug with no repro, no crash dump, no symbols, _no source_…

So think about the show House, MD.  Dr. House is a diagnostician—he diagnoses.  That's his specialty.  He figures out the cases no one can figure out.  Then he hands it off to whatever specialist is appropriate to actually treat it.[^8]

That was the idea.  Impossible bugs would come to me, I'd dig in and figure them out, and then I'd write up my findings and hand them off to the appropriate owning team to actually resolve.  I'd tell them that _X_ was happening while _Y_ was still in progress, which caused _Z_ to break; they'd be in the best position to figure out what to do about that.  Maybe serialize _X_ and _Y_, maybe make _Z_ smart enough to handle the situation, maybe _X_ wasn't even supposed to happen in that situation in the first place… The point being, they know the constraints of their code way better than I do, I'm just the weirdo who came in and somehow pieced together exactly what was happening to cause a problem they may not have even realized was theirs.

It worked pretty well, and I got to see _a lot_ of weird stuff.  That's one of the things I'm planning to do with this blog, is write up some of that weirdness.

# Technologies
- Messed with:
  - Lots of stuff
- Done some actually useful stuff with:
  - SVG
  - Graphviz
  - LaTeX (enough to do my résumé, anyway)
- Substantial amount of work with:
  - C++[^9]
  - C#[^10]
  - JavaScript[^11]
  - Windows batch scripting[^12]
  - MSBuild[^13]

# Other
- I got cats
- Too many Nerf guns
- Old technology rules, I still use my Zune and just put LineageOS on an old Fire tablet I got for 15 bucks at a rummage sale
- I don't use generative AI.  If I'm putting my name on it, I wrote it.  To the best of my knowledge[^14] Everything on this page is, and will always be, 100% human-written.

[^1]: The first SDSU, in South Dakota; not those jokers in California who think San Diego's a State.  Go Jackrabbits.
[^2]: Domino's
[^3]: …and a few short weeks as a ref at a paintball arena…
[^4]: My elbows are still jacked up, factory work is bullshit.
[^5]: Which to a kid from South Dakota, sounded like the biggest number anyone had ever heard of…
[^6]: And then the part of Salesforce that was Tableau, after Tableau got acquired
[^7]: Thanks, Tristan!
[^8]: The other analogy that fits pretty well is Bug Detective, like on Law & Order.  I'd investigate the crime (bug) and then hand it off to the prosecutors (owning team) to resolve.
[^9]: Hate it as only someone who's had to dive in _real_ deep possibly can
[^10]: Love it, favorite language to use, hands-down
[^11]: Mostly causing trouble and weird shenanigans, but _a lot_ of shenanigans
[^12]: The only automation technology I could _count on_ to be available regardless of platform in my time at Microsoft.  I have done orders of magnitude more batch scripting than anyone should ever have had to.  I have scripts that write other scripts
[^13]: I just think it's neat, you know?
[^14]: I.e., modulo any of the innumerable buttholes who are determined to shove LLM garbage into every piece of software in the world managing to sneak some LLM garbage into a tool I use without me realizing it
