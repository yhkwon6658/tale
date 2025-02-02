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
/* 그리드 레이아웃 */
.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr)); /* 최소 200px, 화면 크기에 따라 자동 조정 */
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

/* 이미지 스타일 */
.gallery-item img {
  height: 200px; /* 높이 고정 */
  width: auto; /* 너비는 이미지 비율에 따라 자동 조정 */
  object-fit: cover; /* 높이에 맞게 이미지 조정 */
  border-radius: 10px;
  box-shadow: 0px 4px 6px rgba(0, 0, 0, 0.2);
  transition: transform 0.3s;
}

.gallery-item img:hover {
  transform: scale(1.1);
}

/* 텍스트 스타일 */
.gallery-item p {
  font-size: 14px;
  color: #555;
  margin: 8px 0 0;
}

.gallery-item p strong {
  font-size: 16px;
  color: #333;
}
</style>
