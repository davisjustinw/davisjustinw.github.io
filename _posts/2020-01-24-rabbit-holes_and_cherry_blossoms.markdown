---
layout: post
title:      "Rabbit-holes and Cherry Blossoms"
date:       2020-01-24 03:35:56 -0500
permalink:  rabbit-holes_and_cherry_blossoms
---

## A Javascript / Rails Word Game

I was feeling a bit reflective, so I chose to build a kind of meditative word game for my Javascript/Rails project.  While wandering the internet seeking inspiration, I started researching the origins of haiku.  I stumbled on an interesting paper (If you like to geek out on little morsels of culture and history). Oh glorious internet. 

([RENGA: The Literary Embodiment of Impermanence and Nonself]https://www.uwosh.edu/facstaff/barnhill/244-japan/Renga.pdf by, David Landis Barnhill.

While reading the paper, I imagined stanzas and words fading in and out of existence like another Japanese theme, the cherry blossom.  I grew up in an agricultural area with acres of orchards around.  There's just something about blossoms snowing from their branches.  I wanted to build that.

I dreamed up a game that split up a seed text, then sprinkled words down the screen.  Clicking on the words would add them to running renga verses.  For the text I need something in the public domain.  "Alice's Adventures in Wonderland" just seemed right.

## The Challenges
When I start a project I try to dream up where things could get weird and unpredictable, then rough out plausible solutions.  I identified 3 potential roadblocks

1. The Cherry Blossom Effect
2. Syllable Counting Algorithm
3. Managing EventListeners

Back to the Internet!

Let's talk a little about the Cherry Blossom Effect, some cool resources I found, how I organized my project and some lessons I learned about animation and the bind method.

## The Cherry Blossom Effect
I knew I would need some sort of game loop and that I would have to manage my event handlers well because I'd have thousands of words falling down the screen.  In my research I found a cool site [Kirupa](https://www.kirupa.com/) focused on animation and games that had a [nice article](https://www.kirupa.com/html5/the_falling_snow_effect.htm) on a falling snow effect.  Kirupa also had some nice articles on [creating buttery smooth animations](https://www.kirupa.com/html5/creating_buttery_smooth_animations.htm).  Wait there's more, Kirupa had yet another article on [the requestAnimationFrame method](https://www.kirupa.com/html5/animating_with_requestAnimationFrame.htm) for creating an animation/game loop.

The Javascript project prompt required an object oriented approach with classes.  The Kirupa examples used a prototypal approch with global functions and variables.  This was an awesome learning opportunity to understand the crux of Kirupa's code and adapt it to my needs.  This exercise really forced me to wrap my head around scope and the elusive bind method.

My project was made up of three major classes,

1. Board - The game controller and animation loop, holds objects and dom variables
2. Queue - Keeps a fresh supply of words and moves words around the dom
3. Word - Holds syllable counts, text, and calculates position.

### Cascading State
The Board object passes itself to the Queue and subsequently to Words. I did this to keep variables used in more than one place centralized (DRY).  

If a class higher up in the hierarchy holds information then the child classes call methods in the higher class to then trickle information down.  For example when a Word realizes it has fallen off the screen it calls a destroy and replace me method on the board class which then tells the queue what to do. 

I chose this so I only had to pass the board down to other objects and not a bunch of other variables.  I wasn't super strict about this methodology but I found myself refactoring often to this approach in order to keep things straight.  I was roughly inspired by [Flux architecture](http://fluxxor.com/what-is-flux.html) and React.

### Fun With Bind
My lesson in the bind method was interesting.  Through research I discovered the requestAnimationFrame(callback) method for creating the game loop.  Here is my game loop from the Board class.

```
moveWords() {
   if(!this.paused) {
	    for (const word in this.queue.words) {
			   this.queue.words[word].update();
			}
		} else if(this.paused && !this.timer) {
		   this.playback();
			 this.timer = setInterval(this.playback.bind(this), 3000);
		} 
		
		this.checkForReset();
		requestAnimationFrame(this.moveWords.bind(this));
}
```

As you can see moveWords() passes itself to requestAnimationFrame().  If we are unpaused we update the positions of all the words in the queue.  The requestAnimationFrame() method calls the moveWords() method at a rate that maintains a good framerate for smooth animation.  In the Kirupa article, their callback was global so requestAnimationFrame() could call it until it's heart's content.  When I first took a crack at this, Javascript balked saying, "hey you can't moveWords on undefined."

Something tickled the back of my mind and said, "Hey I've seen this bind method before that was really confusing in a similar context maybe I need to cehck that out."  Bind is one of those things I'd seen used, was super confused about, so just accepted.  I hate those.  So I returned to oh glorious internet to take bind off the just accepted list.  

What does bind do?

Bind passes context.  That's it.  It sends this down another level.  Bam, that's what I needed.  My moveWords() method references this to lookup variables.  When requestAnimationFrame() calls moveWords(), that instance of moveWords() doesn't know what this is.  

So you have to tell it.

So you bind(this) to say hey moveWords(), not to worry, I got your this right here.  You can see I used the same technique on my playback() callback for when the game is paused.  I'm sure I'll forget exactly when to use bind.  But I've commited to memory, bind passes context.  

So remember, if your context is fouled up, maybe reach for bind.

