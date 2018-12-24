---
layout: single-page
---

<h1 style="text-align:center">Final Deliverable</h1>
<h1 style="text-align:center">Eli Lipsitz & Vivien Nguyen</h1>
<hr>

## Abstract

Our project was to create a 2D interactive particle sandbox modeled after popular online falling sand/powder games. Our simulation includes a number of different types of particles (called 'elements') that have unique properties and interact with each other in a variety of ways. The goal of our project was not to create a highly (visually or physically) accurate simulation, but rather to create something with a lot of variety, fast (realtime interactive), interesting to watch, and most importantly, fun to play with.

<br>

## Technical Approach

Each particle is stored separately as structs in a linked list. Each particle struct contains a vector for its 2D position, 2D velocity, element type, and additional information (element type dependent). While each 2D position was a real number, rendering and collision happened on a uniform (1x1 pixel resolution) grid. This collision grid was cached in memory in a 2D array (each position contained a pointer to the particle or a NULL pointer if it was empty), which allowed for constant time collision / reaction lookups.

<br>

There was a separate 'Element' class that stored information about the properties of a specific element: density, air drag, type (solid, liquid, gas), and two important functions. First, `getColor`, which is called by the renderer for each particle --  having a function instead of a constant variable allowed water and fire to change color depending on their velocity, simulating foam and heat. This approach is in general more flexible, especially if true heat flow were implemented. Second, `doUpdate`, which is called by the simulator once per frame per particle: this allows for per-element reactions (fire burning oil, lava and water 'neutralizing').

<br>

In the beginning, the pixel buffer was held in memory, cleared each frame, and rendered directly to the screen with `glDrawPixels`. Later in the project, we switched to using textures and GLSL shaders (and updating to modern-ish OpenGL 3.4 from 2.1). We didn't take full advantage of this flexibility; however, we were really easily able to apply all sorts of image processing effects. If we had more time, we would have implemented a content-aware blur to blend together the same type of element (but not blur across multiple types of elements). 

<br>

In the middle of the project, we also switched from CGL to Nanogui. We did this to allow for an easy GUI (our original control scheme / switching between different elements, 'pen sizes', pausing/resuming, etc. was just a number of difficult-to-remember keyboard commands). This ended up being quite a challenge though, and essentially required gutting a large amount of our user input and screen handling (plus OpenGL context and window creation) code. 

<br>

Airflow simulation was done non-rigorously. Rather than a full implementation of the Navier-Stokes equations, we instead used a 'common sense', empirical model for air. We split the sandbox into 4x4 pixel cells to reduce the number of calculations that we needed to do per frame. Each cell had an associated air pressure and velocity. A difference in pressure between two cells applies a force, which changes the velocity of the air in the cells, and velocity moves pressure between cells. Finally, a Gaussian blur was applied to the pressure grid to smooth out high-frequency pressure changes (which caused unrealistic looking ripples). The end result is an easy, fast, and impressive looking (even if not necessarily physically accurate) air simulation. Since the current state of each grid is dependent only on the previous state (and not the current state of other grid-cells), this could easily be turned into an extremely fast shader. However, in the current version, these computations are done entirely on the CPU.

<br>

One big challenge was implementing proper motion and collision of particles. Extreme care had to be taken to make sure the particles could move freely in a realistic looking way, while preserving collisions between particles. All of the basic particle collision code was developed with Newton's laws of motion, playing around with other similar simulations, and a huge amount of trial and error. Unfortunately, the actual particle motion code is not very parallelizable: since two particles cannot move into the same place at the same time, the simulation relies on the state of all particles at all times, which requires a sequential linear simulation.
<br>


Overall, we weren't quite able to implement as many of the features as we expected to be able to. Neither of us were extremely familiar with C++, and figuring out the OpenGL API calls was a bit of a challenge (and difficult to debug). Things ended up being more difficult, having more problems, or just taking longer than we expected.

<br>

## Results

<div class="row">
<div class="col">
<h2>Poster Session Video</h2>
  <iframe width="480" height="320" src="https://www.youtube.com/embed/DnpGS-dWzyU" frameborder="0" allowfullscreen></iframe>
</div>
<div class="col">
<h2>Final Compilation Video</h2>
  <iframe width="480" height="320" src="https://www.youtube.com/embed/-kABhrsXMNk" frameborder="0" allowfullscreen></iframe>
</div>
</div>


## References

Our primary references were not papers or algorithms, rather, we referenced the games and apps we were trying to emulate. In particular:

* [Powder Game #1](https://dan-ball.jp/en/javagame/dust)
* [Powder Game #2](https://dan-ball.jp/en/javagame/dust2/)
* [The Powder Toy](http://powdertoy.co.uk/)
* [wxSand](http://www.piettes.com/fallingsandgame/)
<br>


## Contributions

Eli primarily worked on the internal simulation engine (implementing the motion and collision of particles).

Vivien primarily worked on adding elements, the GUI, and creating many of the project deliverables.

We both worked together on setting up the initial frameworks and defining the structure of the project (data structures, Particle vs. Element classes, and interaction-simulation-rendering loop). And of course, we both did a _lot_ of debugging.
