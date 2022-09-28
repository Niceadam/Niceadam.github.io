---
layout: post
title: "Vim key remapping thoughts"
---

Here is a common vim problem: you want to insert some code/text you just yanked. 
In the process of making some space for your new code, you accidentally delete a character and
overwrite your register. Now you have to go back and copy your text again.

Destroying your yank when deleting a character is silly. Prefixing delete
commands with `"_` yanks the text into the black hole register
and keeps your yank safe (I use Neovim so my keymaps are in Lua):

{% highlight lua %}
keymap("n", "x", '"_x', opts)
{% endhighlight %}

Its quite often that I want to delete without yanking, creating a separate "mode" can make things easier.
The `s` key is in a convenient position on the keyboard and isn't all that useful
(deletes character and enters insert mode), instead it can be used as a no-yank `d`
(`x` is another option that makes sense, you can map deleting characters to `z`):

{% highlight lua %}
keymap("n", "ss", '"_dd', opts)
keymap("n", "sw", '"_diw', opts)
keymap("n", "s{", '"_d{', opts)
keymap("n", "s}", '"_d}', opts)
keymap("v", "s", '"_d', opts)
{% endhighlight %}

Same thing with `c`, very rarely do I want to yank text that I'm replacing:

{% highlight lua %}
keymap("n", "cc", '"_cc', opts)
keymap("n", "cw", '"_ciw', opts)
keymap("n", "c{", '"_ci{', opts)
keymap("n", "c(", '"_ci(', opts)
keymap("n", "c[", '"_ci[', opts)
{% endhighlight %}

Notice that I mapped to the "around the cursor" version of the commands instead.
I find it more convenient to have yanking and deleting words under my cursor as default.

{% highlight lua %}
keymap("n", "dw", "diw", opts)
keymap("n", "yw", "yiw", opts)
{% endhighlight %}

Recently I was dabbling in other modal editors, like [Helix][helix], and i liked how easy line selection was.
I use visual line and block mode quite often so i dedicated some letters for them too.

{% highlight lua %}
keymap("n", "q", "<S-v>", opts) -- Visual Line
keymap("n", "t", "<C-v>", opts) -- Visual Block
{% endhighlight %}

[helix]: https://helix-editor.com/
