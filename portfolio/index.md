---
layout: default
---

<div class="main">
<div class="page-style-box">
  <h2>Welcome to my portfolio.</h2>
  <p>
  Thank you for taking the time to view my portfolio. The works presented here represent many of the largest projects I've worked on. Projects done for CS 184, Foundations of Computer Graphics, include in depth, technical writeups on the contents of the project. For a quick look of what I've done, you can check out <a href="">my Demo Reel here [last created February 2018]</a>.</p>
</div>
<br>


  <div class="row">
  {% for fp in site.projects %}

    <div class="col" style="margin-bottom:15px">
      <div class="card page-style h-100" >
      <img class="card-img-top" src="{{ fp.img | prepend: site.baseurl }}" alt="Thumbnail">
        <div class="card-body d-flex flex-column">
          <h5 class="card-title">{{ fp.name }}</h5>
          <p class="card-text scroll-box">{{ fp.short }}</p>
          <a href="{{ fp.link | prepend: site.baseurl }}" class="mt-auto btn btn-primary">See More</a>
        </div>
      </div>
    </div>
    {% assign mod = forloop.index | modulo: 3 %}
    {% if mod == 0 or forloop.last %}
      </div><div class="row">
    {% endif %}

  {% endfor %}
  </div>

</div>
