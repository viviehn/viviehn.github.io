---
layout: single-page
---

  <h1 style="text-align:center">Implementation of Animating Pictures <br>with Stochastic Motion Textures</h1>
  <h2 style="text-align:center;margin-bottom:15px">Vivien Nguyen || cs194-26-afd</h2>
  <h2 style="text-align:center;margin-bottom:15px">Original Paper by Yung-Yu Chuang et. al. 2005</h2>
  <hr>

  <h2>Overview</h2>
  <p>In this project, our goal is to take a single static photograph or painting, and animate pieces of the image to make it look alive. We can accomplish this in a system consisting of three main steps: layer generation (segmentation, matting, inpainting), motion specification, and rendering. You can <a href="paper.pdf">read my paper write up here!</a></p>

  <h2>Layer Generation: Segmentation</h2>
  <p><span class="image right"><img src="support/clouds_trimap1.png"></span>
  <strong>Goal:</strong> Select a region to be the "foreground element" of this layer, i.e. the piece to be animated on this layer.
  <br>
  <strong>Process</strong>: Use photoshop to brush in a foreground region (white), background region (black), and ambiguous or unknown region (gray).
  <br>
  <strong>Results</strong>: A trimap, such as to the right.</p>
  <br>
  <br>
  <h2>Layer Generation: Matting</h2>
  <p><span class="image left"><img src="support/mapestimate.png"><figcaption><em>Equation image sourced from Chuang's paper.</em></figcaption></span>
  <strong>Goal</strong>: To recomposite the image, we want the foreground to look natural, even if it is moved to a different location on the background. In order to accomplish this, we will compute an alpha matte and most likely foreground only color for the pixels in the unknown region (gray areas of the trimap).
  <br>
  <strong>Process</strong>: Following "A Bayesian Approach to Digital Matting" by Yung-Yu Chuang et. al. 2001, we will alternately solve the MAP estimation for the most likely Foreground/Background color, then the most likely alpha value, given the observed pixel color for each pixel in the unknown region.
  <br>
  <strong>Results</strong>: A predicted foreground, background, and alpha matte.
  </p>

  <span class="image fit"><img src="support/bayes.png"></span>
  <figcaption align="middle">Why do alpha matting? <br> L: Naive, R: Matted</figcaption>
  <br>
  <h3>Results from Bayesian Matting</h3>
  <div class="box alt">
  <div class="row uniform">
  <div class="4u">
  <span class="image fit"><img src="support/fg_new.jpg"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="support/bg_new.jpg"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="support/alpha_new.jpg"></span>
  </div>
  </div>
  </div>
  <br>

  <h2>Layer Generation: Inpainting</h2>
  <p><span class="image right"><img src="support/inpainting.png"><img src="gifs/inpainting.gif"></span>
  <strong>Goal</strong>: Now that we've segmented and matted our foreground, we'd like to continue segmenting new layers. However, we have a huge black spot in our background. Moreover, when we animate the foreground, it will reveal new parts of the background. So we'd like to inpaint the missing parts!
  <br>
  <strong>Process</strong>: We can use a pretty straightforward inpainting algorithm, "Region Filling and Object Removal by Exemplar-Based Image Inpainting", by Criminisi et. al. 2003. The key element of this method is the fill order: we first compute the "fill front", or the edges of the remaining target region. Then, we compute high priority places to inpaint -- regions that are nearby existing structures in the source region, or regions with high gradients. Then, we define an NxN patch around this point, and search through all same sized patches in the source region. We look for the most similar patch (in the pixels available in the target patch), then copy over the needed pixels.
  <br>
  <strong>Results</strong>: An inpainted background, that we continue to use to segment new layers.</p>
  <br>

  <h2>Motion Specification</h2>
  <p><strong>Goal</strong>: We now have N layers, with a clear foreground element. In order to animate these layers, we need to define motions for each of them. We'd like to compute a displacement map, which in this implementation is a function on the time step <em>t</em>, that returns an [x, y] shift.
  <br>
  <strong>Process</strong>: We bucket our layers into four types of motion - water, boats, clouds, and plants. All of these are sums of sines (with the exception of clouds, which is a simple translation), but applied in different ways. For water and boats, we use the same sine sum within the same image, with the phase offset dependent on the y pixel of the pixel to be translated. This creates a nice rippling effect. We ask the user to click a point at the bottom of the boat, so we can set it in phase with the nearby water. We can also use similar techniques for even non-boat layers, like the circles example below. For that one, we use a dampening sine function to create a bouncing effect.
  <br>
  For plants, we ask the user to define a line running through the plant from up to down. Then, we map all the y values into a [0,1] range relative to this user-defined line. After generating our sinusoid, we apply it using linear interpolation across these y values.
  <br>
  <strong>Results</strong>: A function that we can call at each time step to generate a new frame.</p>
  <span class="image fit"><img src="support/rendering.png"></span>
  <figcaption align="middle">What we have now</figcaption>

  <br>
  <h2>Rendering and Results</h2>
  <p>The final step of our pipeline is to render our frames! We proceed with layers from back to front (painter's algorithm) to synthesize each frame of our animation. At each time step, we call the displacement function for each layer, add that shift, and recomposite the layers using our computed alpha mattes.</p>
  <br>

  <div class="box alt">
  <div class="row uniform">
  <div class="4u">
  <h4>The Boat Studio, 1876, Claude Monet</h4>
  </div>
  <div class="4u">
  <span class="image fit"><img src="inputs/boatstudio.jpg"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="gifs/boatstudio.gif"></span>
  </div>
  </div>

  <div class="row uniform">
  <div class="4u">
  <h4>Sailing Boats - Morning, 1926, Yoshida Hiroshi</h4>
  </div>
  <div class="4u">
  <span class="image fit"><img src="inputs/sailingboat.jpg"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="gifs/sailingboat.gif"></span>
  </div>
  </div>

  <div class="row uniform">
  <div class="4u">
  <h4>Several Circles, 1926, Wassily Kandinsky</h4>
  </div>
  <div class="4u">
  <span class="image fit"><img src="inputs/circles.png"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="gifs/circles.gif"></span>
  </div>
  </div>

  <div class="row uniform">
  <div class="4u">
  <h4>Above the Clouds, 1929, Yoshida Hiroshi</h4>
  </div>
  <div class="4u">
  <span class="image fit"><img src="inputs/clouds.png"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="gifs/clouds.gif"></span>
  </div>
  </div>

  <div class="row uniform">
  <div class="4u">
  <h4>Duck Pond, <a href="https://community.glwb.net/gallery/index.php/Lorain-County-Summer-Photos/Duck-Pond-Fall-4">Source</a></h4>
  </div>
  <div class="4u">
  <span class="image fit"><img src="inputs/ducks.png"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="gifs/ducks.gif"></span>
  </div>
  </div>

  <div class="row uniform">
  <div class="4u">
  <h4>Sunflower Field, <a href="https://www.readingeagle.com/news/article/elverson-sunflower-field-a-sunny-spot-in-a-sometimes-dark-world">Source</a></h4>
  </div>
  <div class="4u">
  <span class="image fit"><img src="inputs/sunflowers.png"></span>
  </div>
  <div class="4u">
  <span class="image fit"><img src="gifs/sunflowers.gif"></span>
  </div>
  </div>

  </div>
