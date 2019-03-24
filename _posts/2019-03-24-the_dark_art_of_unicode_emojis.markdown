---
layout: post
title:      " The Dark Art of Unicode Emojis "
date:       2019-03-24 22:41:59 +0000
permalink:  the_dark_art_of_unicode_emojis
---


## Not Just for Ninjas
My first memory of [unicode emojis](https://unicode.org/emoji/charts/emoji-list.html) used in a terminal was while installing [Homebrew](https://brew.sh/).  A sting of jealousy even inadequacy washed over me as the little beer mug &#127866; appeared on my terminal.  "How'd they do that?" I thought. Certainly only Ninja Magician Hackers have the brains, the tools, and the hubris to show frosty beverages on my terminal.  Funny how a simple emoji on a terminal seems so cool.  Or maybe it's just me.

Many Flatiron instructors and students spread love by rocking terminals with a cream colored background and a heart emoji &#10084;.  Ninjas everywhere.  I learned about unicode escape codes by trying to figure how to edit my bash profile to show a rabbit prompt &#128007;. I'm an unapologetic Matrix fan and have Alice's disposition for plunging into rabbit holes.

> "The first, often overlooked step, to solving problems; Believing, no knowing, you can find the way."

On a [recent ruby project](https://github.com/davisjustinw/luned), I combined weather and lunar phase data with Seattle 911 data. The [weather api](https://darksky.net/poweredby) I was polling, returns a decimal value for lunar phase.  In a command line interface, the decimal value didn't tell the story I wanted, so I turned to unicode emojis.

To display emojis as part of strings in ruby output you use the escape character '\u' followed by the hexidecimal value for the emoji in braces.  I find [this website](https://unicode.org/emoji/charts/emoji-list.html) helpful for looking up codes. To print the hedgehog to terminal would look something like this:

```
puts "\u{1F994}"

```
=>&#129428;

## My Lunar Solution

From class Luned::View
```
require 'rounding'

...

def moon(phase)
  # takes a float between 0 and 1, returns unicode escape for corresponding moon phase emoji.

  icons = {"0.0"=>"\u{1F311}", "0.125"=>"\u{1F312}", "0.25"=>"\u{1F313}",\
	      "0.375"=>"\u{1F314}", "0.5"=>"\u{1F315}", "0.625"=>"\u{1F316}",\
		  "0.75"=>"\u{1F317}", "0.875"=>"\u{1F318}", "1.0"=>"\u{1F311}"}

  icons[phase.round_to(0.125).to_s]
end
```

There are eight unicode moon emojis but the Dark Sky API returns lunar phase as a decimal between zero and one. The moon method initializes a hash with a unicode escape code for every multiple of 0.125 between zero and one.

I used the rounding gem's round_to method to round the float parameter (phase) to the nearest eighth (0.125).  Hashes don't take numeric keys so I convert to string before looking up the rounded value in the icons hash.

```
moon(0.5)
=> "\u{1F315}"

```
Which printed to the terminal will look like... &#127765;

Cheers! &#127866;


