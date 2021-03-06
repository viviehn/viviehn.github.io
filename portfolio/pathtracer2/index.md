---
layout: single-page
---

<h1 style="text-align:center">CS 184: Computer Graphics and Imaging, Spring 2017</h1>
<h1 style="text-align:center">Project 3-2: Ray Tracer, Part 2</h1>
<h1 style="text-align:center">VIVIEN NGUYEN, CS184-acd</h1>
<hr>


<div class="row">
<div class="col">
  <img src="images/dragon_mirror_1024_64_100.png" class="img-fluid" />
  <img src="images/lucy_256_16_100_1440.png" class="img-fluid" />
</div>
<div class="col">
  <img src="images/pt2_bunny.png" class="img-fluid" />
  <img src="images/dragon.png" class="img-fluid" />
</div>
</div>
<br>

<h2 align="middle">Overview</h2>
<p>In this project, I extended the path tracing program I created in the previous part to simulate even more visual effects. In particular, I added support of three more types of BRDFs/materials (mirror, glass, and microfacet), implemented environment lights from an environment map as a light source, and added the ability to change focal length/lens radius to simulate a thin lens and create depth of field effects.</p>
<br>

<h2 align="middle">Part 1: Mirror and Glass Materials</h2>
<p>For this part, we implemented two new materials: ideal mirrors, and glass. Implementing mirror materials was relatively simple, largely relying on the reflect function. We simply reflect the wo vector around the normal vector (0,0,1) and store that in wi; we set pdf to 1.0 since we did not actually do any sampling, and we return the reflectance divided by the cosine factor to account for later multiplying by a cosine factor (since perfect mirrors do not exhibit light fall off).</p>
<br>
<p>Glass materials are slightly more complicated, and require the refraction function. This function takes in a wo vector and returns a wi direction after refraction, which is dependent on the index of refraction of the materials the ray is passing through (in this case, air and glass). In the refract function, we can use Snell's Law to determine if we have total internal reflection (only possible if the ray started inside the object). If that is the case, we can return false and we don't store a wi direction. </p>
<br>
<p>Then, in the sample_f function for glass, we essentially have three cases. In the case of total internal reflection, we simply store wo in wi and set pdf to 1.0. However, if there isn't total internal reflection, we have the option of refracting or reflecting; the probability of which we use Schlick's approximation to calculate. Schlick's reflection coefficient gives us R, so we reflect with probability R and refract with probability 1-R. Depending on the result of coinflip(R), then, we perform the correct transformation and assign the corresponding probability to pdf.</p>
<br>
<div class="row">
<div class="col">
    <img src="images/pt1_spheres_0.png" class="img-fluid" />
    <figcaption align="middle">With 0 bounces, we only have direct lighting, and these materials haven't been able to "reveal" themselves yet.<br><br><br><br><br><br><br></figcaption>
     <img src="images/pt1_spheres_1.png" class="img-fluid" />
     <figcaption align="middle">With one bounce, the mirror ball displays most of the features we'd expect. One thing to note is that it takes one extra bounce for the mirror ball to reveal features of the box - note that the ceiling in this render is already white due to global illumination, but this hasn't been reflected in the mirror ball yet. The glass ball looks a lot like the mirror ball currently, since the only things we can see from it are bounces that were immediately reflected according to Schlick's reflection coefficient.</figcaption>
</div>
<div class="col">
    <img src="images/pt1_spheres_0_rate.png" class="img-fluid" />
<br>
     <img src="images/pt1_spheres_1_rate.png" class="img-fluid" />
</div>
</div>
<div class="row">
<div class="col">
    <img src="images/pt1_spheres_2.png" class="img-fluid" />
    <figcaption align="middle">Here, we can see that the ceiling has appeared the correct color in the mirror ball, but the reflection of the ball doesn't look right - it's reflecting what the ball looked like in the previous render!<br><br></figcaption>
     <img src="images/pt1_spheres_3.png" class="img-fluid" />
     <figcaption align="middle">Notice the caustic at the bottom of the glass ball - this took the exact amount of bounces we would expect it to appear. One bounce for light to hit the glass ball, one for it to refract inside the ball, and one more for it to refract back out and show light.</figcaption>
</div>
<div class="col">
    <img src="images/pt1_spheres_2_rate.png" class="img-fluid" />
<br>
     <img src="images/pt1_spheres_3_rate.png" class="img-fluid" />
</div>
</div>
<div class="row">
<div class="col">
    <img src="images/pt1_spheres_4.png" class="img-fluid" />
    <figcaption align="middle">At this point, the mirror ball has mostly converged, nothing to note other than it looks quite a bit brighter at the bottom of the ball - this is just a result of more reflecting bounces between the ball and the floor. The glass ball still has a lot of interesting new effects though, namely the spot of light it's now created on the blue wall.</figcaption>
     <img src="images/pt1_spheres_5.png" class="img-fluid" />
     <figcaption align="middle">Not too many changes to note between 4 and 5 bounces, but in general we can say that the glass ball seems to get increasingly "noisier" - why? Because as we increase the number of bounces, we get a wider variety of refractions and reflections, which gives us more complicated lighting effects. <br><br></figcaption>
</div>
<div class="col">
    <img src="images/pt1_spheres_4_rate.png" class="img-fluid" />
<br>
     <img src="images/pt1_spheres_5_rate.png" class="img-fluid" />
</div>
</div>
<div class="row">
<div class="col">
    <img src="images/pt1_spheres_100.png" class="img-fluid" />
    <figcaption align="middle">At 100 bounces, or Russian Roulette termination, we have two pretty intricate looking balls. The lighting effects on the glass ball, in particular, are less "blocky" and look a lot more natural. <br><br><br></figcaption>
</div>
<div class="col">
     <img src="images/spheres_1024_64_100_1440.png" class="img-fluid" />
     <figcaption align="middle">The previous images were rendered at 256 samples per pixel and 4 samples per light; here's one at 1024 samples per pixel and 64 samples per light. Probably the largest effect is the increased focus of the caustics on the ground and on the blue wall from the glass ball.</figcaption>
</div>
</div>
<p>As a final note, it was brought to my attention that supposedly the second, less bright rectangle on the glass ball is not actually expected behavior. It seemed natural to me - a result of the light from the top rectangle being reflected inside the ball and then refracted back out. Perhaps there was some minor numerical error, or that these effects are actually supposed to be "invalid" and killed off (sad).</p>
<br>
<h2 align="middle">Part 2: Microfacet Material</h2>
<p>This part relied on implementing three main functions: one to calculate the Normal Distribution Function, one to calculate the Fresnel term, and one to importance sample the BRDF. There were also two functions used calculate the shadow masking term and to evaluate the BRDF, but these were already implemented/relatively trivial.</p>
<br>
<p>As the name suggests, the normal distribution function defines the distribution of the microfacets' normals. Only microfacts whose normals lie exactly on the halfangle vector of wi and wo can reflect wi to wo (we assume the microfacts are ideal specular). We use the Beckmann distribution with regards to the halfangle vector to figure out our normal distribution. This function also queries alpha, which sets the "roughness" of our surface - higher alpha means our surface is more diffuse or matte, and lower alpha means our surface is glossier (looks kind of mirror-like).</p>
<br>
<p>The Fresnel term is calculated at R, G, and B wavelengths because calculating it for every possible wavelength would be very very hard. We can do pretty good just by calculating it for those three wavelengths using eta and k values for our desired conductor/metal.</p>
<br>
<p>Finally, we want to implement importance sampling for our BRDF, to replace the cosine hemisphere sampling method currently in the function body. We want to sample with priority on pdfs resembling the normal distribution function, because that'll give us less noise and quicker convergence. We can use the inversion method, then, to get a pdf that resembles the Beckmann NDF. For some solid angle, we want to sample both a theta and a phi value, and combine the two. This gives us a sampled halfangle vector, which allows to have something to reflect wo by, and we can plug the resultant relevant values into the BRDF evaluation function.</p>
<br>
<div class="row">
<div class="col">
  <img src="images/pt2_spheres.png" class="img-fluid" />
  <img src="images/pt2_bunny.png" class="img-fluid" />
</div>
</div>
<div class="row">
<div class="col">
  <img src="images/pt2_gold_005.png" class="img-fluid" />
  <figcaption align="middle">alpha = 0.005; Quite noisy, not just on the model but on the surrounding walls of the box itself! Very shiny/glossy, almost mirror like.</figcaption>
  <img src="images/pt2_gold_05.png" class="img-fluid" />
  <figcaption align="middle">alpha = 0.05; Still hard to distinguish from the mirror material, but the very bright spots of noise are less noticeable</figcaption>
  <img src="images/pt2_bunny_unif.png" class="img-fluid" />
  <figcaption align="middle">Bunny with uniform cosine hemisphere samplining; plenty of noise and practically no convergence.</figcaption>
  <img src="images/pt2_platinum_005.png" class="img-fluid" />
  <figcaption align="middle">Platinum dragon at alpha = 0.005</figcaption>
  <img src="images/pt2_platinum_25.png" class="img-fluid" />
  <figcaption align="middle">Platinum dragon at alpha = 0.25</figcaption>
</div>
<div class="col">
  <img src="images/pt2_gold_25.png" class="img-fluid" />
  <figcaption align="middle">alpha = 0.25; Starting to look a lot more diffuse/rough, and the noise is almost eliminated at this point. <br></figcaption>
  <img src="images/pt2_gold_5.png" class="img-fluid" />
  <figcaption align="middle">alpha = 0.5; Very diffuse and rough looking, can see the individual glinty-ness of the material quite nicely.</figcaption>
   <img src="images/pt2_bunny_imp.png" class="img-fluid" />
   <figcaption align="middle">Bunny with importance samplining; still a bit of noise but we have a much better idea of the actual material of the bunny (copper).</figcaption>
   <img src="images/pt2_platinum_05.png" class="img-fluid" />
   <figcaption align="middle">Platinum dragon at alpha = 0.05</figcaption>
   <img src="images/pt2_platinum_5.png" class="img-fluid" />
   <figcaption align="middle">Platinum dragon at alpha = 0.5</figcaption>
</div>
</div>
<br>

<h2 align="middle">Part 3: Environment Light</h2>
<p>The first two sections of this part were relatively simple and mostly just involved sampling the environment map for its radiance at a particular location. Uniform sampling meant that we sample a random direction on the sphere and check for the environment map's radiance in that direction.</p>
<br>
<p>However, we want to implement importance sampling for environment lights as well - we want the real-world effect of energy being concentrated in the direction of bright lights. So, we want to be able to bias our sampling towards such directions. This decreases the amount of noise in our renders dramatically and allows for much quicker convergence - like most other importance sampling methods, it's a "smarter" way of choosing samples, rather than uniformly at random.</p>
<br>
<p>While the motivation for importance sampling is simple, I found it quite challenging in this particular case due to having many steps/portions to achieve the correct result. First, we calculate the pdf for every index i, j of the environment map, where the pdf for each index is dependent on the radiance value at that spot - as expected, since we want the probability of sampling that location to be relative to the brightness of that location. Next, we compute the marginal distribution p(y), which is approximately the cumulative distribution function for sampling any particular row of the environment map. The third step is to compute the conditional distribution, which is approximately the cumulative distribution function for sampling any particular 2D index, given the row. We use Bayes' rule here to store the conditional distribution. Finally, we adapt our sample function to utilize these three calculated arrays to do our "smarter" importance sampling!</p>
<br>

<div class="row">
<div class="col">
  <img src="images/field.png" class="img-fluid" />
</div>
<div class="col">
  <img src="images/probability_debug.png" class="img-fluid" />
</div>
</div>
<div class="row">
<div class="col">
  <img src="images/pt3_bunny_unlit_unif.png" class="img-fluid" />
  <figcaption align="middle">Diffuse, uniform sampling, field.exr. Noisy, and very little convergence - it's hard to see any of the details of the bunny figure, or its shadow.</figcaption>
  <img src="images/pt3_bunny_unlit_cu_unif.png" class="img-fluid" />
  <figcaption align="middle">Microfacet, uniform sampling, field.exr. It doesn't look too noisy or bad at all, until you compare it to the importance sampled version, which has way more intricate highlights, shadows, etc.</figcaption>
  <img src="images/pt3_bunny_unlit_unif2.png" class="img-fluid" />
  <figcaption align="middle">Diffuse, uniform sampling, grace.exr</figcaption>
  <img src="images/pt3_bunny_unlit_cu_unif2.png" class="img-fluid" />
  <figcaption align="middle">Microfacet, uniform sampling, grace.exr</figcaption>
</div>
<div class="col">
   <img src="images/pt3_bunny_unlit_imp.png" class="img-fluid" />
   <figcaption align="middle">Diffuse, importance sampling, field.exr. Looks a lot better! The bunny model looks less flat, we can see both the shadow on the platform as well as the shadows on the model.</figcaption>
   <img src="images/pt3_bunny_unlit_cu_imp.png" class="img-fluid" />
   <figcaption align="middle">Microfact, importance sampling, field.exr. Way more visual effects - we can actually tell this is a shiny metal, versus the very matte appearance of the uniformly sampled bunny.</figcaption>
   <img src="images/pt3_bunny_unlit_imp2.png" class="img-fluid" />
   <figcaption align="middle">Diffuse, importance sampling, grace.exr</figcaption>
   <img src="images/pt3_bunny_unlit_cu_imp2.png" class="img-fluid" />
   <figcaption align="middle">Microfacet, importance sampling, grace.exr</figcaption>
</div>
</div>
<br>
<p>I wanted to include the second set of environment images (the bunny in grace.exr) because I was honestly so surprised about what a dramatic effect the envrionment map could have on the model - it's basically an entirely different "color" than the field.exr bunny, simply because it's appearance is so reliant on the amount and placement of lights in the environment map, as well as the dominating colors of the environment.</p>
<br>

<h2 align="middle">Part 4: Depth of Field</h2>
<p>Up this point, we've been working with a "pinhole" camera simulation - everything in the view of camera is perfectly in focus. However, this is not how real cameras, nor how the human eye, truly works. Instead, we implement thin-lens simulation to more accurately simulate the visual effects we are used to.</p>
<br>
<p>The ideal thin-lens has no actual thickness. However, it still can simulate the refractive effects we desire. Our sensor plane/the screen can now receive light from any location, not just the (0,0,0) center - which was the case when we had a pinhole camera. However, we can use the pinhole camera ray (generate_ray) to figure out where the light will hit the sensor plane. We know this will be the point in focus, and we instead just calculate the ray that goes to that point but from a randomly sampled point on the lens.</p>
<br>
<p>This part was relatively simple, but I spent a non-trivial amount of time debugging why rotating my camera put my scene in some corner of the window. I was mistakenly blindly copying the code from generate_ray to create the ray that went through the center of the lens, which was operating in world, not camera space. Relocating the origin of this ray to be specifically (0,0,0) solved that issue, and I could now create great in/out of focus effects!</p>
<br>

<div class="row">
<div class="col">
  <img src="images/pt4_dragon_1024_02.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 0.2, aperture: 0.03125</figcaption>
  <img src="images/pt4_dragon_1024.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 1.2, aperture: 0.03125</figcaption>
  <img src="images/pt4_dragon_1024_72.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 7.2, aperture: 0.03125</figcaption>
  <img src="images/pt4_dragon_1024_0.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 1.2, aperture: 0.0</figcaption>
  <img src="images/pt4_dragon_1024_4.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 1.2, aperture: 0.044194</figcaption>
  <img src="images/pt4_dragon_1024_8.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 1.2, aperture: 0.088388</figcaption>
  <img src="images/pt4_dragon_1024_5.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 1.2, aperture: 0.5</figcaption>
</div>
<div class="col">
   <img src="images/pt4_dragon_1024_05.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 0.5, aperture: 0.03125</figcaption>
   <img src="images/pt4_dragon_1024_32.png" class="img-fluid" />
   <figcaption align="middle">Focus length: 3.2, aperture: 0.03125</figcaption>
  <img src="images/pt4_dragon_1024_132.png" class="img-fluid" />
  <figcaption align="middle">Focus length: 13.2, aperture: 0.03125</figcaption>
   <img src="images/pt4_dragon_1024.png" class="img-fluid" />
   <figcaption align="middle">Focus length: 1.2, aperture: 0.03125</figcaption>
   <img src="images/pt4_dragon_1024_6.png" class="img-fluid" />
   <figcaption align="middle">Focus length: 1.2, aperture: 0.062500</figcaption>
   <img src="images/pt4_dragon_1024_25.png" class="img-fluid" />
   <figcaption align="middle">Focus length: 1.2, aperture: 0.250000</figcaption>
   <img src="images/pt4_dragon_1024_1.png" class="img-fluid" />
   <figcaption align="middle">Focus length: 1.2, aperture: 1.0</figcaption>
</div>
</div>

