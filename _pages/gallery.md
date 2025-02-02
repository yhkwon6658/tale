---
layout: post
title: Gallery
author: "Yonghwan Kwon"
permalink: /gallery/
---

<div class="gallery">
{% for item in site.data.gallery %}
  <div class="gallery-item">
    <img src="{{ item.image }}" alt="{{ item.title }}">
    <p><strong>{{ item.title }}</strong></p>
    <p>{{ item.description }}</p>
  </div>
{% endfor %}
</div>

<style>
.gallery {
  display: flex;
  flex-wrap: wrap;
  gap: 20px;
  justify-content: center;
}
.gallery-item {
  width: 200px;
  text-align: center;
  margin: 20px;
}
.gallery-item img {
  width: 100%;
  border-radius: 10px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s;
}
.gallery-item img:hover {
  transform: scale(1.1);
}
</style>
