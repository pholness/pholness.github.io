---
title: Coding
has_children: false
nav_order: 2
---
# Coding

<details open markdown="block">
  <summary>
    Table of contents
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Although not exclusively about coding, as I'm keeping some of my source code in github, it makes sense to keep the notes on it here too...
## C++
The language I spent most of my software dev career using. I've come back to C++ after several years away, and have been playing with SDL2 and C++ on Linux
### The Barnsley Fern Generator.
This project is a simple implementation of the [Barnsley Fern fractal](https://en.wikipedia.org/wiki/Barnsley_fern)

The code creates an SDL2 window and renderer, then uses a helper object to calculate points on the fern, drawing them with the `SDL_RenderDrawPoint` function. The code updates the rendered dots 60 times per second. It takes a couple of seconds to generate a few hundred thousand points.

The binary accepts a numeric argument that caps the number of points to calculate. If no parameter is given it defaults to 100,000.

The output looks like this.

![Fern Image](/assets/images/fern.png)

### Lines1

A hark back to the screen savers of old. 

A simple demo that draws lines bouncing around the screen.

* Q to Quit
* Esc to clear the screen
* C to change the movement of the end points.

It looks like this.

![Lines1 Image](/assets/images/Lines1.png)

### Lines2

An improved hark back to the screen savers of old. 

A simple demo that draws lines bouncing around the screen. This time it keeps track of the last 100 lines, and deletes the oldest line for each frame.

* Q to Quit
* Esc to clear the screen
* C to change the movement of the end points
* Up Arrow to slow down
* Down Arrow to speed up.

It looks like this.

![Lines2 Image](/assets/images/Lines2.png)

### Mandelbrot Set Renderer

Something I've often seen, but never tried writing.

The application lets you click to centre and zoom the image. The max number of iterations can be passed as a parameter to the program (more iterations takes longer to render, but lets you zoom in to far greater depth)

* Q to Quit
* R to reset to the lowest zoom level
* Z to zoom in without changing the centre point
* Left mouse click - re-centre on click point and zoom in.

It looks like this.

![Mandelbrot Image](/assets/images/Mandelbrot.png)


## Python

Not much to see here yet.

## Raspberry Pi

Likewise, but something is coming...

