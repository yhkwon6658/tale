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

<!-- 팝업 HTML -->
<div id="popup" class="popup">
  <div class="popup-content">
    <span class="popup-close" onclick="closePopup()">&times;</span>
    <h2 id="popup-title"></h2>
    <p><strong>Name:</strong> <span id="popup-name"></span></p>
    <p><strong>Date:</strong> <span id="popup-date"></span></p>
    <p><strong>Technology:</strong> <span id="popup-technology"></span></p>
    <p><strong>Die Area:</strong> <span id="popup-die-area"></span></p>
    <p><strong>Core Area:</strong> <span id="popup-core-area"></span></p>
    <p><strong>Performance:</strong> <span id="popup-performance"></span></p>
    <p><strong>Power:</strong> <span id="popup-power"></span></p>
    <p><strong>Information:</strong> <span id="popup-info"></span></p>
  </div>
</div>

<style>
/* 그리드 레이아웃 */
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 30px;
  justify-content: center;
}

/* 팝업 스타일 */
.popup {
  display: none;
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.7);
  justify-content: center;
  align-items: center;
  z-index: 999;
}

.popup-content {
  background: #fff;
  padding: 20px;
  border-radius: 10px;
  width: 400px;
  max-width: 90%;
  text-align: left;
}

.popup-close {
  position: absolute;
  top: 10px;
  right: 20px;
  font-size: 24px;
  font-weight: bold;
  cursor: pointer;
}
</style>

<script>
function showPopup(title, name, date, technology, dieArea, coreArea, performance, power, information) {
  document.getElementById('popup-title').innerText = title;
  document.getElementById('popup-name').innerText = name;
  document.getElementById('popup-date').innerText = date;
  document.getElementById('popup-technology').innerText = technology;
  document.getElementById('popup-die-area').innerText = dieArea;
  document.getElementById('popup-core-area').innerText = coreArea;
  document.getElementById('popup-performance').innerText = performance;
  document.getElementById('popup-power').innerText = power;
  document.getElementById('popup-info').innerText = information;

  document.getElementById('popup').style.display = 'flex';
}

function closePopup() {
  document.getElementById('popup').style.display = 'none';
}
</script>