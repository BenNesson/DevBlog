+++
date = '2025-06-18T15:06:05-07:00'
draft = true
title = "A Study in Scarlet (but it's a Bug)"
tags = [ "Bug Detective", "Dr. Nesson", "Windows Guts", "Security", "QtWebEngine" ]
summary = "Season 1, Episode 1…"
+++

## Early June, 2021[^1].
Extracts[^2] were failing.  Specifically, WebDataConnector[^3] extracts on Tableau Server, and more specifically, they were failing:
- on the newest version
- when sandboxing was enabled[^4]
- when the run-as user was an Administrator
- because one of the QtWebEngine worker processes was crashing for some reason

[^1]:
    This was actually the very first official Dr. Nesson investigation.  Which is to say, this wasn't my _first_ impossible investigation, but it was my first investigation _as a full-time bug detective_.  Seemed like a good one to start the blog with.

[^2]:
    Extracts are basically a way to pull just the data needed for a viz or dashboard and bundle it up into an independent package for the viz or dashboard to point to.  This means the viz doesn't need to hit the actual original datasource (which can be a huge amount of work for more complicated vizzes), but it also means if the original data changes, your extract is out-of-date.  So Extracts are typically configured to refresh periodically.  That refresh is what was failing.

[^3]:
    WebDataConnectors are a nifty technology that allow for custom connectors to be written in html/javascript.

[^4]:
    Sandboxing is a WebEngine/chromium thing where the child processes QtWebEngine uses run with greater restrictions, for security purposes.

I was looped in relatively early, both because I had experience dealing with WebEngine[^5] and because crash investigations were a particular specialty of mine (there's so many tools to collect information from them, and those artifacts always have _so much more_ information than anyone seems to realize).  And because it was getting hot, so we needed a solve _fast_.  As I got spun up on the situation, all the initial pieces were there to lay out the overarching story, they just needed to be put in a coherent order: when the protocol server used QtWebEngine to run the WDC code, something was causing one of the QtWebEngine child processes to crash; the protocol server detected that crash, and failed out of the WDC extract as a result.  In other words, the protocol server was working fine, the problem was that crashing child process.

[^5]:
    I did the initial prototyping of the Hybrid Dialog/Hybrid UI system on Desktop, which was basically a way to use a little browser window to host html/javascript, allowing reuse of UI built for the web version of the product on desktop.  I then co-led the team that implemented the feature, focusing personally on the interop layers that allowed the html/javascript side to communicate and interact with the native backend.  (Last I saw, most of that core interop code was still in place, largely unchanged.)

So, why was the child process crashing?

### The Easy(ish) Part
I started digging into the child process, initially by way of procmon logs that had already been collected.  We actually had a repro on this one (which was not typically the case for my investigations), but it required setting up Tableau server, which was an arduous enough process that I wanted to avoid it if (or, at least, as long as) possible.  And procmon logs contain a lot of stack information, so it was worth looking through what we already had to find a jumping off point.

I tracked down the last event in the child process before all the ThreadExit events started firing and it suggested, oddly enough, an issue trying to find _fonts_.  The last couple stack frames before everything went all system-exception-handling were `blink::FontFallbackIterator::Next` calling into `blink::FontCache::CrashWithFontInfo`.  Since `blink` is part of chromium's layout code, it looked like during layout, the process was trying to access fonts, failing, and then deciding the only option remaining was to crash.

This was a little odd, because during WDC extract refresh, there's nothing to actually render or display.  The process essentially runs headless, with no inputs from/outputs to a user.  So this crash was happening because chromium couldn't do a thing that we didn't care if it did in the first place.  But that's chromium for you, it's built with a lot of assumptions about how it's going to be used, and we… didn't always fit those assumptions.

Confirming this theory introduced the first wrinkle, though.  I was still hoping to avoid setting up server on my machine (and I've done a lot of my best work with memory dumps[^6]), so I asked someone who was already set up with a repro to point procdump at the child process and… the bug stopped happening.

[^6]:
    As will become _very apparent_ as I share more of these stories…

If procdump was attached, the child process wouldn't crash.

### The Game is Afoot
No-repro-under-debugger is always _a treat_.  Now, I have a tendency to anthropomorphize the bugs I'm working on, because I find it's effective motivation.  I find emotional attachment can be a good way to engage subconscious/non-rational/intuitive parts of the brain, which are often the source of the most significant breakthroughs.  So whenever a bug I'm hunting shows off a trick like this, it's a little like when one of my cats sees a bunny outside move.  _Now_ I'm engaged.  _Now_ I'm hunting for my food.

It turns out, this was (I believe) my first encounter with _breakpoint exceptions_.  If you know JavaScript, the idea is a little like the `debugger` statement there, which will break into the debugger if one is attached, _and do nothing otherwise_.  I emphasize the "_do nothing otherwise_" part because that's _not_ how [`DebugBreak()`](https://docs.microsoft.com/en-us/windows/win32/api/debugapi/nf-debugapi-debugbreak) works.  Let's go to the docs:
> If the process is not being debugged, the function uses the search logic of a standard exception handler. In most cases, _this causes the calling process to terminate because of an unhandled breakpoint exception_.  \[_Emphasis added_\]

So this is a thing where, if a debugger is attached, it'll harmlessly break in; if no debugger is attached, it _hauls off and crashes the whole damn process_.  It's basically instant different-behavior-under-debugging!  I hope I never meet whoever thought this was a good idea, for liability reasons!

Anyway, the reason the repro didn't work with procdump attached was because _procdump_ was actually handling the exception.  And without the exception, everything else ran fine[^7].  Fortunately, it's relatively straightforward to tell procdump to stop handling those breakpoint exceptions and treat them like the unhandled crashing exceptions they actually are.  This allowed me to get an actual memory dump of the actual crash when it actually happened, which... didn't really tell me a whole lot, it turns out.

[^7]:
    Apparently, anyway.  But judging by the function name `blink::FontCache::CrashWithFontInfo`, this was pretty clearly not a situation where chromium code expected things to continue on, so it seemed reasonable to consider anything after that point to be undefined behavior, semantically if not syntactically.

At least, not directly.  Digging in and debugging the code in question clarified something: with the sandbox enabled, the child process wasn't trying to access fonts directly itself, it was requesting them from the parent process (the protocol server).  And the bug didn't repro with sandboxing disabled.  So clearly the problem was somewhere in that interaction.

### Some Processes' Parents…
Going back to the procmon logs backed this up.  With the sandbox enabled, the parent process showed a bunch of file access events for a bunch of fonts—an Open, Query, and Close event for each.  With the sandbox _disabled_, the parent process didn't get involved at all.  The child process accessed the fonts directly, and _only had to open the one before proceeding_.  This pretty much shouts that the problem is that the parent can't open files for the child.  I dug into the IPC[^8] code a bit to understand how it worked, and after a bit more debugging determined that the parent was in fact reporting failure every time the child asked it to open a file.

[^8]:
    Inter-process Communication, this is just a general term for code that lets processes talk to each other directly.

So it turns out the problem was with the protocol server after all.  I started debugging that, and in short order had the biggest chunk of the puzzle so far: `NtCreateFileInTarget`.  This function is (as I recall) part of the chromium IPC code used by the sandboxing system.  It's the function on the parent-side that handles opening files for the child process.  It tries to do three things:
  - Open the file in question
  - Confirm the file opened was the file actually requested
  - Hand the file handle over to the child process that asked for it.

Somehow, we were getting hung up on the second step.  Hence, the Open-Query-Close events from the parent: It opens the file fine, then does some kind of querying to confirm it got the right file, but that query fails somehow, so the file is closed and the parent tells the child it couldn't open the right file.

More specifically, I discovered that, when `NtCreateFileInTarget` used another internal function `SameObject` to confirm that, e.g., `C:\Windows\Fonts\somefont.ttf` and `\Device\HarddiskVolume3\Windows\Fonts\somefont.ttf` were actually the same path, `SameObject` was calling a Windows API function called `QueryDosDeviceW` to get the device/volume name for `C:`.  That is, `SameObject` needed to know whether `C:` and `\Device\HarddiskVolume3` corresponded to the same thing.  So it would call `QueryDosDeviceW`, passing it the name of the device (`C:`) and a string to be populated with that device name, and it expected the length of that string to be returned from the function.

But `QueryDosDeviceW` wasn't returning the length of the string, it was returning 0, which indicates an error.  `SameObject` didn't bother checking that error, but I get aggressive with a debugger, so I did: `ERROR_ACCESS_DENIED`.

So, somehow the parent process was able to open the file, but was _not_ able to query the device name for the root drive.  _Weird._  But like I said, this was a _massive_ chunk of the puzzle solved.  This weird access/permissions issue meant the parent couldn't open any files for the child, so the child couldn't access fonts, so it threw that breakpoint exception, the child process crashed, and the protocol server registered the failure and aborted the extract.  So what was causing the access issue?

At this point, the next important clue that had been there all along was that the bug would only repro if the Run-As User was an administrator.  When you first set up Tableau Server, it'll ask you what user to actually run the server as.  You can have it run as the default Network Service account, but there's any number of reasons you might want to create a specific user to run things as instead.

Making that user an Administrator was a simple short-cut to ensuring it had a number of permissions it'd need for things to work, so it was a pretty common configuration.  Of course, running everything from an elevated Admin profile can be dangerous[^9], so we had always taken steps to make sure some things ran restricted instead.

[^9]:
    _Especially_ browser stuff, and since the QtWebEngine-based WDC extract was running in a little chromium browser, it was _especially_ important to ensure it wasn't running with elevated Admin rights.

One of those things was the protocol server.

### I Don't (Need to) Know What I'm Doing
To go any deeper, I was clearly going to need to figure out the whole restricted security token situation.

I hate security stuff.  Security Tokens, Certificates, Access Control Lists, Permission Sets…  It's always tended to be pretty impenetrable, and never explained in a way that has stuck for me.

Fortunately, one of the weirdest things about how implausibly good I am at bug investigations is how important me actually understanding the stuff involved _isn't_.  I always seem to figure out exactly as much as necessary to understand how the pieces aren't fitting together.

So I made a pair of _Things_: a _Parent_ Thing and a _Child_ Thing.  The Parent Thing created a restricted security token, and used it to launch the Child Thing.  The Child Thing called `QueryDosDeviceW` the same way the chromium code running in the protocol server did[^10].  Then I tracked down the code we used to launch restricted processes, looked it over (understanding very little of it), and duplicated enough of it into the Parent Thing that I could successfully replicate the same failure in the Child Thing that was causing our problems in the protocol server.

[^10]:
    There's potential for confusion here, because until now I've been referring to the protocol server as the parent and the QtWebEngine worker process as the child.  But the parent-child relationship I was trying to analyze at this point in the story was between the zookeeper (the central coordinating process that manages all the other various processes that make up Tableau Server) as parent, and the protocol server as child.
    
    The zookeeper ran elevated and launched the protocol server with a restricted token.  That mechanism was somehow causing the permissions issue that was causing `QueryDosDeviceW` to fail.  That's the breakdown I needed to understand.

The reason I made these Things is that they were a _hell_ of a lot easier to debug and rev on than a whole entire server installation.  I didn't have to change the run-as user's permissions and restart the whole server, I could just run the Parent Thing from an elevated/non-elevated command prompt.  That ease of revision let me determine that out of the three things we did to make a Restricted Security Token:
  - Deny a couple user groups, including the Administrators group
  - Disable all privileges except one
  - Some kind of DACL thing I never understood (and never needed to)

The only one that mattered was denying the Administrators group.  If the Parent Thing didn't disable those privileges, the problem still happened.  If it _didn't_ do the DACL thing, the Child Thing wouldn't work at all.  But denying the Administrators group meant the problem occurred, and _not_ denying it meant the problem _didn't_.  And in either case, running the Parent Thing from a non-admin command prompt meant the bug didn't repro.

### The Bottom of the Rabbit Hole
What was happening when the parent process was elevated, but the child process wasn't?  All I was getting from `QueryDosDeviceW` was `ERROR_ACCESS_DENIED`, which was not especially informative.  Searching online did little to illuminate anything, because we were pretty deep into territory that most folks steer clear of, and pretty far off the rails as far as what Microsoft has actually documented.

So, deeper still.  Via some aggressive debugging and disassembling, I started poking around inside `QueryDosDeviceW`.  What I found was that one of the first things it does is call `NtOpenDirectoryObject` to open `\??`, which is a path prefix that represents kind of the root of all device objects on the system.  But importantly, the Windows Object Manager translates `\??` into something different on a per-user, per-session, per-lots-of-stuff basis, including _whether the process token is elevated_.

What this means is that a program that calls `NtOpenDirectoryObject` on `\??` will be pointed at a _different object_ when run elevated than when run non-elevated.  And importantly, if an elevated parent creates a security token that denies the Administrators group and uses it to run a child, and that child calls `NtOpenDirectoryObject` on `\??`, it gets _the same object_ as the elevated parent gets with its unrestricted token.

In other words—and this is the very heart of the bug—the Windows Object Manager _still sees the Restricted token as an elevated token_.

This is the crux of things because, most importantly, looking up the paths in question in WinObj and examining their Access Control List?  The path from the non-elevated process includes the user account in its ACL, but the path from the elevated process _doesn't_.  That ACL only includes _the Administrators group_.

So.  This is the root.  With this, we have all the pieces we need to know the bug's True Name[^11].

[^11]:
    The True Name of a bug is what it is, how it happens—the elements involved and how they interact to create the bug.  To know a bug's True Name is to understand its exact nature.  Knowing a bug's True Name grants one tremendous power over it, typically (but not always) including the power to destroy it.

### The True Name
When the Run-As User for a service is an Administrator, Windows launches that service process elevated.  Always.

The Tableau Server Zookeeper service, running elevated, tries to launch the protocol server with a non-elevated security token.  Windows doesn't really provide a way to do this, though.  So the protocol server actually runs with an elevated security token that doesn't have access to the Administrators group.

The protocol server spins up the WDC protocol.  QtWebEngine spins up a renderer child process to run the WDC's JavaScript/HTML.  The renderer child process, trying to render the HTML, requests fonts from the parent process.

The parent process opens each font file, then tries to confirm the file opened and the file requested are the same.  When that verification code tries to confirm the device name of the root drive, the device lookup gives it an object specific to the elevated user account.  That object's Access Control List only allows that user account access via the Administrators group.  The Administrators group has been denied to the process's security token, so access is denied.  This causes the root drive device lookup to fail, which causes the same-object verification to fail.

The protocol server reports to the renderer child process that it could not open the requested font file.  The renderer child process runs out of fonts to try to open, and crashes.  The protocol server registers this crash and aborts the extract.

### The Fix
There really wasn't one.  At least, not one in code.

Ultimately, it was kind of a fluke that it worked before.  The reason we hadn't been hitting the bug previously is because WebDataConnectors had previously been implemented with QtWebKit instead of QtWebEngine.  And WebKit just didn't happen to call `QueryDosDeviceW` on `C:`, so this never came up.

As far as resolving via a code change, we really didn't have a lot of options.  If the Run-As User for a service is an Admin, Windows _will_ run that service elevated.  And there really just _isn't_ a way for an elevated process to get a non-elevated token.  You can try cloning one off of a non-elevated process, but there's no clean way to _find_ a non-elevated process to use for that, or ensure one is running.  You can't do it via a service, that's just begging the question.  And you can't assume the Run-As User is actually logged into the machine and running explorer.exe or anything.

In the end, this really just ended up acting as a forcing function for us updating our guidance to recommend _against_ using an Administrator account as the Run-As User.  This was something we had apparently been moving toward for some time anyway; it's not a great security practice, and so far as I could determine was really only something we ever suggested because it was a quick shortcut to giving the Run-As User the various permissions it needed.  But over the years, we'd refined and adjusted enough other things that it really wasn't necessary, and it remaining part of the guidance was basically an artifact.

### Conclusions
I feel like this one went pretty well as a first run of the whole Bug Detective concept.  It had a lot of the hits: obscure Windows guts, SysInternals tools, disassembling OS code to rip answers out of it…  Not to mention, the first appearance of Breakpoint Exceptions!  And between the security token stuff, the Windows Object Manager stuff, and the Tableau Server stuff, it had plenty of the "I'm only as much of an expert in this as I've needed to become over the last two or three days' worth of digging" that tended to be a hallmark of so many of these investigations.

Turnaround time wasn't too bad considering the depth of the problem.  Subtracting a couple weeks for a birthday trip and wrapping up some other unrelated work, it was maybe three or four weeks of investigation from the beginning of the fire drill until the cause was understood and we were talking about resolutions, and another two or three weeks from there until we were turning out the lights on it.

Difficulty-wise: not the most impossible solve, to be honest.  Tough, but doable.  I'm sure plenty of subject matter experts would have known that that deny-the-Admin-group trick wouldn't work and would cause weird access errors, but that's definitely a flavor of "hindsight is 20/20".

And not especially weird.  Technical and esoteric, sure, it's a good candidate for Doc-Brown technobabbling at someone.  But it doesn't have a real "Wait, _what?_" moment in it like I always love to get.