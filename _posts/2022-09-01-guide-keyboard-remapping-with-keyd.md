---
layout: post
title: "primer on keyboard remapping"
---

Becoming a programmer means typing a lot of symbols outside of the usual letter set and moving around with the arrow keys.
This usually involves using shift with the number row, and moving your hand all the way to the arrow keys.
Most programmers accept this input system and don't think twice about it. I was one of those people..
until i saw [Neal Wu's typing][nealwu]. My mind couldn't grasp how this guy was shifting + number row and using the arrow keys at 100+ WPM.
The only way I could come up with to produce symbols that fast and smoothly was if he had the same level of access to
the symbols as with the normal letter set. Everything has to be one row away from the home row 
(I'll call this the home zone). How can that be possible?

# Layers
A layer is a modifier key, that when held remaps the entire keyboard to a custom character set or macros.
Infact you already use a layer on a regular basis, the `Shift` key is a layer. It transforms all lowercase
letters to uppercase. Layers is what allows to map our far away keys onto the home zone.

# AltGr
The thumb is our most agile finger, and yet it has access to very little keys. Most of the space down there
is taken up by the spacebar (pun partially intended), of which only the vicinity of one spot is actually pressed most of the time.
But the thumb does have access to the `AltGr` key, which we can use as our symbol modifier. If you're lucky like me,
on some keyboards it is found right below `,` making it relatively easy to access and hold
(If you're left handed, you might want to try Left Alt instead, although that might clash with 
some default shortcuts on your OS). We can now map most of our symbol set using AltGr as a modifier

# Remapping using Keyd
Keyboard remapping of on Linux was a pain not too long ago (`xmodmap`). Now there are some nice
projects around that provide easy ways of managing mappings. I use [keyd][keyd], it does the job wonderfully
and configuration is a relatively simple process. Here is my `/etc/keyd/default.conf`

{% highlight toml %}
[ids]
*

[main]

# mod1 Layer
leftalt = alt
rightalt = layer(mod1)

[mod1]
i = up
j = left
k = down
l = right
u = home
o = end

e = (
r = )
d = {
f = }
c = [
v = ]
q = !
w = #
a = =
s = _

t = *
g = &
b = %
y = ^
h = +
n = $

. = playpause

{% endhighlight %}

The most important map to notice here is `AltGr`+`ijkl` to arrow keys.
This completely changed how I used a computer, because now my movement in any application happens on the home zone (like `hjkl` in Vim).
I mapped all the bracket types to the `edc`+`rfv` columns. I use `{}` as my main big vertical movement in Vim so they are placed right under my left home row.
I even have a map to play/pause any music that is playing.

# Caps Lock
`Caps Lock` is frankly, not that useful, and yet its placed in a great spot for a modifier key,
so lets make it one! Vim users (such as myself) might already be familiar with mapping their
Caps Lock to Esc, that doesn't stop us from making another layer. Keyd supports overloading, which allows
a button to act differently based on if its just pressed or held. We can set Caps Lock to Esc when pressed
and activate a layer when held.

{% highlight toml %}

# Maps capslock to escape when pressed and mod2 when held.
capslock = overload(mod2, esc)

# Remaps the escape key to capslock
esc = capslock

[mod2]
u = 7
i = 8
o = 9
j = 4 
k = 5
l = 6
m = 1
, = 2
. = 3
n = 0
y = y
h = -

d = <  
f = >
a = \
s = |
{% endhighlight %}

I mapped some more symbols and a keypad.

# Alt
Alt is another layer I use for application control like opening apps, managing Alacritty
and executing Tmux commands. This gives me a contextual separation between the layer keys. Note I use KDE.

{% highlight html %}
Alt-Q: Switch to or open Alacritty
Alt-F: Switch to or open Firefox
Alt-w: Switch to or open Teams

Alt-<Arrow-key>: Navigate Tmux panes
Alt-<HJKL>: Resize Tmux panes
Alt-s: Browse Tmux sessions 
{% endhighlight %}

For the 'switch to or open' functionality, I used `wmctrl`. For example, `wmctrl -a 'Firefox' || firefox` switches to a window with 'Firefox' in the title,
or if such a window is not found, opens a new instance.

# Conclusion
I'm still figuring out efficient ways to map out all of my input into the computer. Holding AltGr for long periods may cause strain although I haven't yet experienced any.
Since my right thumb also is my spacebar finger, it can cause a speed bottleneck when typing alot of interweaved symbols and spaces. Currently I'm learning to incorporate my left
thumb to do the spacebaring in some of these situations, but this is where the limitations of the normal keyboard are felt. The normal physical layout was just not designed to be used
to such a customized extent, and now would be a good idea to start looking at more [esoteric keyboards][keyboards].

[keyd]: https://github.com/rvaiya/keyd
[nealwu]: https://www.youtube.com/watch?v=LK7tDnzUZZM
[keyboards]: https://www.youtube.com/watch?v=SkNGxM4LRKQ
