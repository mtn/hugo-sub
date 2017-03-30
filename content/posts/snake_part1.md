+++
draft = true
date = "2017-03-28T21:39:31-05:00"
title = "Snake with GNU ncurses! -- Part I, ncurses basics"

+++


As my first game, I made snake in C using the graphics library ncurses! There are probably several dozen other languages that would have also sufficed or been better but I chose C because I picked it up pretty recently, so making a project from scratch was a decent challenge. Over two posts, I'll walk through some of the process, including decisions I made along the way and the rationale behind them. In this one, I'll set up the basic features of ncurses that I found useful. What I write should be _mostly_ self-contained, but when included links should clear things up in cases where I'm not.

# What's ncurses?

ncurses is probably the easiest graphics library to get started with (not that I have much other experience yet), and is very well documented [here](http://tldp.org/HOWTO/NCURSES-Programming-HOWTO/). It came about as a successor to curses, a wrapper over terminal graphics functionality that allows us to render pretty things on different terminals without working with messy raw values. To the best of my knowledge, the necessary header is included with the Xcode developer tools for MacOS, and is available through `apt-get` (or your default package manager) on linux. To make something simple like snake, there are only a few key concepts you need to wrap your mind around. Starting simple is good; I first stumbled on ncurses after googling "C terminal graphics" when working on a (not-yet-finished) Conway's Game of Life simulator, and felt I was making too many decisions because I wanted to progress in the project without getting a feel for the full library. There's lots more, though, which I'll definitely be exploring in the future and you should check out after reading this post too!

## Initialization Functions

To initialize our ncurses session, we first call `initscr()`. There are also a few functions we call at the start our program in order to define basic behaviors:

`cbreak()` -- Disables line buffering and also allows us to process control sequences (eg. CTRL^C)

`noecho()` -- Turns off input echoing, as its name suggests :). This means that the keys you press will not be printed on the screen like they normally are when you type into the terminal

`keypad(stdscr,TRUE)` -- Allows us to enable processing of additional keys like the function keys and more importantly, the arrow keys (unless you like to play snake with WASD or something, I guess). Note the first argument, `stdscr` can be varied if we want to do the same thing for windows, which I'll talk about soon

`curs_set(0)` -- Hides the cursor, which we wouldn't want distracting from our beautifully designed game ;)

`halfdelay()` --  Sets an upper bound on the amount of time the program waits for input (after which it returns ERR). Sounds useful, right?

## Output and Input

ncurses essentially gives you a grid systems of rows and columns. Within this basic grid, we can do simple things like move around and print characters but, as we'll see, also introduce more additional structures. As with many similar programs, the top left corner of the screen is (0,0) rather than the bottom left as is conventional elsewhere. What is rather more unique is that throughout ncurses where coordinate arguments are used, the "y" is specified before the "x." A few functions!

### Our Output Functions

`move(y,x)` -- Allows us to move the cursor to a specified location on the screen. Note that if we hide the cursor (call `curs_set(0)`), this doesn't produce any externally-observable change

`addch(ch | ATTRIBUTES)` -- Adds a character at the current location of the cursor. Note that we can OR [attributes](http://www.tldp.org/HOWTO/NCURSES-Programming-HOWTO/attrib.html)

`mvaddch(y, x, ch)` -- A composition of the prior two functions

`printw(str)` -- The `printf` of ncurses

`mvprintw(y, x, str)` -- Similar to `mvaddch`, a composition of `move` and `printw`

### Our Input Functions

`getch()` -- Reads a character from the terminal. Note that unless `cbreak` or `raw` are called, input will be newline-buffered

And actually, that's the only one! There are certainly others with fairly self-explanatory names, such as `scanw`, but snake is relatively simple and we won't be needing them :).

## A Few Others...

There are a few other functions that we'll find useful. One of these is `getmaxyx(stdscr,y,x)`, which allows us to get the dimensions of our current screen (or window, if we alter the first parameter). We can use these numbers to calculate the coordinates we'll render elements of our interface.

## Windows

When we call `initscr`, we are given a clean canvas to work with -- the entire terminal. Windows allow us to add additional "layers" to this canvas, which is useful if we want to manage separate parts of the display independently. Example cases would be the game itself or a score display, which we might want to update multiple times per second asynchronously. There are a few special functions, though on the whole windows behave mostly in a similar way to the screen itself:

`newwin(height, width, starty, startx)` -- Initializes a new window at the coordinates `(starty,startx)` relative to `stdscr`

`delwin(win)` -- Deletes a window, deallocating its memory

Apart from these two functions, windows can mostly be manipulated by functions named after their standard counterparts with a "w" appended to the start. For example, `printw` becomes `wprintw`. The first argument to all these functions is the some window.

## Refresh

Congrats! You're well on your way to snake in ncurses. But if you've tried anything out yet, you'll have realized there's a problem. Maybe you have the wrong version...? No! I've just left the most important part for last :). On this last point, it's helpful to think of a virtual screen that exists separately from your physical screen. Changes we make go into effect immediately on the virtual screen -- if we print something to the screen, for example -- but we need to push them onto the physical screen somehow. This is where `refresh()` comes in (and correspondingly, `wrefresh`)! Whenever we make a change, we need to _refresh_ our screen in order to see it in real life. When in doubt, check if you've done this.

## That was a long list...

Wow, that was a lot! Don't worry if you can't remember more than a few. As you start to work with ncurses, you'll quickly internalize them. Until then, I hope this will be a good reference. Before moving on, it's worth making a simple program that uses each of the functions above. If you're feeling adventurous, poke around for other functions in the [HOWTO](http://www.tldp.org/HOWTO/NCURSES-Programming-HOWTO/attrib.html) and try out some others. Good luck!

