# GitHub Pages site for Peter Holness
## Introduction
My intention with this site is to keep notes on some of the silly little projects I am playing around with at any one time. The hope being that when I come back to a toy project I was working on in the past, I will have some clue about what I was doing when I parked it.
## Coding
Although not exclusively about coding, as I'm keeping some of my source code in github, it makes sense to keep the notes on it here too...
### C++
The language I spent most of my software dev career using. I've come back to c++ after several years away, and have been playing with SDL2 and c++ on Linux
#### The Barnsley Fern Generator.
This project is a simple implementation of the [Barnsley Fern fractal](https://en.wikipedia.org/wiki/Barnsley_fern)
The code creates an SDL2 window and renderer, then uses a helper object to calculate points on the fern, drawing them with the `SDL_RenderDrawPoint` function. The code updates the rendered dots 60 times per second. It takes a couple of seconds to generate a few hundred thousand points.
The binary accepts a numeric argument that caps the number of points to calculate. If no parameter is given it defaults to 100,000.
The output looks like this.
![Fern Image](/assets/images/fern.png)
#### Lines1
A hark back to the screen savers of old. 
A simple demo that draws lines bouncing around the screen.
* Q to Quit
* Esc to clear the screen
* C to change the movement of the end points.
#### Lines2
An improved hark back to the screen savers of old. 
A simple demo that draws lines bouncing around the screen. This time it keeps track of the last 100 lines, and deletes the oldest line for each frame.
* Q to Quit
* Esc to clear the screen
* C to change the movement of the end points
* Up Arrow to speed up
* Down Arrow to slow down.
### Python
Not much to see here yet.
### Raspberry Pi
Likewise, but something is coming...
## Databases
A place to keep information about databases, primarily Oracle ones.
### Useful SQL
To be populated.
