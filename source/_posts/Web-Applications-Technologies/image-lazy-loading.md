---
title:  页面图片懒加载机制
description: 页面图片请求太多，影响整个页面的相应速度，采用懒加载机制可以提速。本文采用bootstrap的转灯carousel做为示例
...

页面图片请求太多，影响整个页面的相应速度，采用懒加载机制可以提速。本文采用bootstrap的转灯carousel做为示例

## 非懒加载
如果一个页面比较复杂, 转灯有几十上百个图片，无疑会很影响页面的性能。下面这个是笔者实际经历的，转灯有几十个slides, 图片就更多了，一次性的图片并发相当大，严重影响页面的响应速度。
![转灯延时加载](http://tech.jiu-shu.com/Web-Applications-Technologies/carousel-lazy-loading.jpg)

下面的代码是bootstrap的转灯carousel的示例，通过chrome的开发者工具可以看到，会有三个image的request 发出。
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Bootstrap Example</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>

<div class="container">
  <h2>Carousel Example</h2>  
  <div id="myCarousel" class="carousel slide" data-ride="carousel">
    <!-- Indicators -->
    <ol class="carousel-indicators">
      <li data-target="#myCarousel" data-slide-to="0" class="active"></li>
      <li data-target="#myCarousel" data-slide-to="1"></li>
      <li data-target="#myCarousel" data-slide-to="2"></li>
    </ol>

    <!-- Wrapper for slides -->
    <div class="carousel-inner">
      <div class="item active">
        <img src="images/portfolio-img1.jpg" alt="Los Angeles" style="width:100%;">
      </div>

      <div class="item">
        <img src="images/portfolio-img2.jpg" alt="Chicago" style="width:100%;">
      </div>
    
      <div class="item">
        <img src="images/portfolio-img3.jpg" alt="New york" style="width:100%;">
      </div>
    </div>

    <!-- Left and right controls -->
    <a class="left carousel-control" href="#myCarousel" data-slide="prev">
      <span class="glyphicon glyphicon-chevron-left"></span>
      <span class="sr-only">Previous</span>
    </a>
    <a class="right carousel-control" href="#myCarousel" data-slide="next">
      <span class="glyphicon glyphicon-chevron-right"></span>
      <span class="sr-only">Next</span>
    </a>
  </div>
</div>
<script>
    $(function() {
        $('#myCarousel').on('slide.bs.carousel',  function(ev) {
            var lazy;
            lazy = $(ev.relatedTarget).find("img[data-src]");
            lazy.attr("src", lazy.data('src'));
            lazy.removeAttr("data-src");
        });
    });
</script>
</body>
</html>

```

## 懒加载
通过chrome的开发者工具可以看到进入页面后初始只有第一个图片的请求产生。
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <title>Bootstrap Example</title>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
  <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.3.1/jquery.min.js"></script>
  <script src="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/js/bootstrap.min.js"></script>
</head>
<body>

<div class="container">
  <h2>Carousel Example</h2>  
  <div id="myCarousel" class="carousel slide lazy" data-ride="carousel">
    <!-- Indicators -->
    <ol class="carousel-indicators">
      <li data-target="#myCarousel" data-slide-to="0" class="active"></li>
      <li data-target="#myCarousel" data-slide-to="1"></li>
      <li data-target="#myCarousel" data-slide-to="2"></li>
    </ol>

    <!-- Wrapper for slides -->
    <div class="carousel-inner">
      <div class="item active">
        <img src="images/portfolio-img1.jpg" alt="Los Angeles" style="width:100%;">
      </div>

      <div class="item">
        <img data-src="images/portfolio-img2.jpg" alt="Chicago" style="width:100%;">
      </div>
    
      <div class="item">
        <img data-src="images/portfolio-img3.jpg" alt="New york" style="width:100%;">
      </div>
    </div>

    <!-- Left and right controls -->
    <a class="left carousel-control" href="#myCarousel" data-slide="prev">
      <span class="glyphicon glyphicon-chevron-left"></span>
      <span class="sr-only">Previous</span>
    </a>
    <a class="right carousel-control" href="#myCarousel" data-slide="next">
      <span class="glyphicon glyphicon-chevron-right"></span>
      <span class="sr-only">Next</span>
    </a>
  </div>
</div>
<script>
    $(function() {
        $('#myCarousel').on('slide.bs.carousel',  function(ev) {
            var lazy;
            lazy = $(ev.relatedTarget).find("img[data-src]");
            lazy.attr("src", lazy.data('src'));
            lazy.removeAttr("data-src");
        });
    });
</script>
</body>
</html>

```


