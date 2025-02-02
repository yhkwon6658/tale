---
layout: post
title: Gallery
author: "Yonghwan Kwon"
permalink: /gallery/
---

<div class="gallery">
{% for item in site.data.gallery %}
  <div class="gallery-item">
    <div class="image-container">
      <img src="{{ item.image }}" alt="{{ item.title }}">
      <div class="info-overlay">
        <p><strong>Technology:</strong> {{ item.description }}</p>
        <p><strong>Technology:</strong> {{ item.technology }}</p>
        <p><strong>Type:</strong> {{ item.type }}</p>
        <p><strong>Area:</strong> {{ item.area }}</p>
        <p><strong>Power:</strong> {{ item.power }}</p>
        <p><strong>Performance:</strong> {{ item.performance }}</p>
      </div>
    </div>
    <p><strong>{{ item.title }}</strong></p>
  </div>
{% endfor %}
</div>

<style>
/* 그리드 레이아웃 */
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr)); /* 최소 200px, 화면 크기에 따라 자동 조정 */
  gap: 30px; /* 이미지 간격 */
  justify-content: center;
}

/* 각 아이템 스타일 */
.gallery-item {
  text-align: center;
  display: flex;
  flex-direction: column;
  align-items: center;
}

/* 이미지 컨테이너 */
.image-container {
  position: relative;
  display: inline-block;
}

/* 이미지 스타일 */
.image-container img {
  width: 100%;
  height: 300px; /* 높이 고정 */
  object-fit: cover; /* 이미지 비율 유지 */
  border-radius: 10px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s;
}

.image-container img:hover {
  transform: scale(1.1);
}

/* 오버레이 정보 */
.info-overlay {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.7); /* 반투명 검정 배경 */
  color: white;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  opacity: 0; /* 기본적으로 숨김 */
  transition: opacity 0.3s ease; /* 전환 효과 */
  padding: 10px;
  box-sizing: border-box;
  border-radius: 10px;
}

.image-container:hover .info-overlay {
  opacity: 1; /* 마우스 오버 시 표시 */
}
</style>
