---
title: 相册
layout: page
---

<style>
div.imgs {
  display: flex;
  flex-wrap: wrap;
}
div.imgs .img-container {
    display:inline-block;
    width: 260px;
    height: 260px;
    padding: 10px;
    border: 1px solid #ccc;
    border-radius: 4px;
    box-shadow: 1px 1px 5px 0 #e6e6e6, -1px -1px 5px 0 #e6e6e6;
    margin: 10px;
}
div.imgs .img-container img {
    width:100%;
    height:100%;
}
div.imgs .img-container img:hover {
    transform: scale(1.1);
    transition: transform .6s;
}
</style>

<div class="imgs">
    <div class="img-container"><img src="/images/IMG_0800.JPG" alt="图片文本描述"></div>
    <div class="img-container"><img src="/images/IMG_1301.JPG" alt="图片文本描述"></div>
    <div class="img-container"><img src="/images/IMG_1426.JPG" alt="图片文本描述"></div>
    <div class="img-container"><img src="/images/IMG_1494.JPG" alt="图片文本描述"></div>
    <div class="img-container"><img src="/images/IMG_1497.JPG" alt="图片文本描述"></div>
    <div class="img-container"><img src="/images/IMG_14262.jpg" alt="图片文本描述"></div>
</div>

<div style="clear: left;"></div>
