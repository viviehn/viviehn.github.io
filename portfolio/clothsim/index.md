---
layout: single-page
---

<h1 style="text-align:center">CS 184: Computer Graphics and Imaging, Spring 2017</h1>
<h1 style="text-align:center">Project 4: Cloth Simulator</h1>
<h1 style="text-align:center">Vivien Nguyen, CS184-acd</h1>
<hr>


<h2>Overview</h2>
<p>In this project, I created a real-time cloth simulator using a mass and spring system. With this system, we can discretely represent the cloth to more easily apply physical forces and constraints on them. Using numerical integration, we are able to simulate the way cloth moves under a variety of circumstances - pinned, falling, and collision with objects. We can also implement self-collision handling to avoid the cloth clipping into itself.</p>
<br>

<div class="row">
<div class="col-md-8">
<h2>Part I: Masses and Springs</h2>
<p>
The first step to simulating cloth is to build a physical model we can use for the cloth itself. Here, we represent the cloth using a grid of masses and springs. First, we create all of the masses according to the specified num_width_points and num_height_points. Depending on the orientation of the cloth, we want to keep either y-coords constant or z-coords (relatively) constant. However, for vertical cloths, we want to actually slightly vary z within some random offset - this will allow us to simulate vertical cloths falling in later parts, rather than standing straight up. 
</p>
<br>
<p>
After storing all the masses in row-major order, we want to create springs between certain combinations of masses. In particular, we create structural constraints in a regular grid - between a mass and the one to its left, and between a mass and the one above it. We create bending constraints similarly, but between a mass and the one two to its right, and between a mass and the one two above it. Finally, we create shearing constraints between a mass and the one to its upper diagonal left, and between a mass and the one to its upper diagonal right. 
</p>
</div>

<div class="col">
  <img src="images/mesh.png" class="img-fluid" />
  <figcaption align="middle">One view of the pinned2 cloth mesh</figcaption>
  <img src="images/mesh_2.png" class="img-fluid" />
  <figcaption align="middle">Another view of the pinned2 cloth mesh</figcaption>
</div>
<div class="col">
  <img src="images/shearing.png" class="img-fluid" />
  <figcaption align="middle">Shearing constraints only</figcaption>
  <img src="images/no_shearing.png" class="img-fluid" />
  <figcaption align="middle">Structural and bending constraints only</figcaption>
</div>
</div>
<br>

<div class="row">
<div class="col-md-8">
<h2 align="middle">Part II: Simulation via Numerical Integration</h2>
<p>
Now that we have a cloth model, we want to apply forces on our mass-spring system to simulate motion. First, we need to calculate all the forces acting on each point mass. Forces can be split into external accelerations spring correction forces. After accumulating the external accelerations, we calculate the spring forces using Hooke's law and add a vector with a magnitude equal to the spring force and in the direction of a mass on one end of the spring to the one on the other end (and an opposite force to the other mass).
</p>
<br>
<p>
Once we have the forces for each mass, we want to adjust the actual positions of each mass. We use Verlet integration to approximate the position at this 'upcoming' timestep, given the current timestep's position and the previous timestep's position.
</p>
<br>
<p>
Finally, we want to constrain our updated positions so that our springs don't stretch infinitely! We want to ensure that the spring length is at most 1.1 times its rest_length in order to better simulate a rigid cloth model.
</p>
</div>
<div class="col">
  <img src="images/15g.png" class="img-fluid" />
  <figcaption align="middle">Density at 15g</figcaption>
  <img src="images/2g.png" class="img-fluid" />
  <figcaption align="middle">Density at 2g, less rippley</figcaption>
  <img src="images/30g.png" class="img-fluid" />
  <figcaption align="middle">Density at 30g, more rippley</figcaption>
</div>
<div class="col">
  <img src="images/5000ks.png" class="img-fluid" />
  <figcaption align="middle">5000 ks</figcaption>
  <img src="images/1000ks.png" class="img-fluid" />
  <figcaption align="middle">1000 ks, stretches a lot more</figcaption>
  <img src="images/10000ks.png" class="img-fluid" />
  <figcaption align="middle">10000 ks, a lot stiffer</figcaption>
</div>
</div>
<br>

<div class="row">
<div class="col-md-8">
<h2 align="middle">Part III: Handling Collisions with Other Objects</h2>
<p>
For both the sphere and the plane, we want to calculate collisions with the cloth and these objects, and "bump up" the cloth so that it is very slightly above the object in question. In both cases, we calculate the point where the mass would have intersected the sphere or plane if it had traveled in a straight line from its last position and its current position. </p>
</div>
<div class="col">
  <img src="images/5000ks_sphere.png" class="img-fluid" />
  <figcaption align="middle">5000 ks</figcaption>
  <img src="images/500ks_sphere.png" class="img-fluid" />
  <figcaption align="middle">500 ks, a much looser cloth, with softer drapes that clings closer to the sphere</figcaption>
</div>
<div class="col">
  <img src="images/50000ks_sphere.png" class="img-fluid" />
  <figcaption align="middle">50000 ks, stiffer springs make a stiffer cloth, for example, a vinyl table cloth that doesn't drape smoothly. </figcaption>
  <img src="images/plane.png" class="img-fluid" />
  <figcaption align="middle">Cloth lying flat on top of the plane</figcaption>
</div>
</div>
<br>

<h2 align="middle">Part IV: Handling Self-Collisions</h2>
<p>The final step in this project is to handle self-collisions. Without this implemented, our cloth will just fall through itself as if its not an object itself. A naive solution would be to calculate distances between every pair of masses on each simulate tick, but this would take O(N^2) time for each call to simulate, which is very unreasonable, especially as the cloth increases in size or complexity.</p>
<p>Instead, we employ spatial hashing; a mass's position becomes associated with some value representing its relative position in 3D space. Once we can calculate a unique identifier for any position, we can build a spatial map to narrow down masses that are likely to collide with one another.</p>
<p>Now to each call to simulate, we also look for self-collisions. For any mass, we loop through ones that hash to the same spatial region, as these are the masses likely to be close enough to create a collision. If there are any, we want to move the mass in consideration to some average location away from all the masses it was closest to. </p>
<br>

<div class="row">
<div class="col">
  <img src="images/15g_collide.png" class="img-fluid" />
  <figcaption align="middle">15g, draping down</figcaption>
  <img src="images/2g_collide_1.png" class="img-fluid" />
  <figcaption align="middle">2g, draping down - a lot wider folds</figcaption>
</div>
<div class="col">
  <img src="images/15g_collide_3.png" class="img-fluid" />
  <figcaption align="middle">15g</figcaption>
  <img src="images/2g_collide_3.png" class="img-fluid" />
  <figcaption align="middle">2g, cleaner, wider folds</figcaption>
</div>
<div class="col">
  <img src="images/15g_collide_2.png" class="img-fluid" />
  <figcaption align="middle">15g</figcaption>
  <img src="images/30g_collide_1.png" class="img-fluid" />
  <figcaption align="middle">30g, floppier and messier folds </figcaption>
</div>
<div class="col">
  <img src="images/15g_collide_4.png" class="img-fluid" />
  <figcaption align="middle">15g, folding over itself on the plane</figcaption>
  <img src="images/30g_collide_3.png" class="img-fluid" />
  <figcaption align="middle">30g, wrinkley folds</figcaption>
</div>
<div class="col">
  <img src="images/500ks_collide.png" class="img-fluid" />
  <figcaption align="middle">500ks, cloth is softer and less stiff, so results in more folds</figcaption>
  <img src="images/50000ks_collide.png" class="img-fluid" />
  <figcaption align="middle">50000ks, cloth is stiffer so folds are fewer and wider</figcaption>
</div>
</div>


