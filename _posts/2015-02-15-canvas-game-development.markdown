---
layout: post
title: "HTML5 Canvas游戏开发心得"
categories: jekyll update
date: 2015-02-15 16:49:30
tags: canvas, performance optimization
---

#目录

[SetTimeout or RequestAnimationFrame](#settimeout-or-requestanimationframe)

[Control your Game FPS](#control-your-game-fps)

[Canvas Size](#canvas-size)

[Multiple Canvases](#multiple-canvases)

[OffScreen Canvas](#off-screen-canvas)

[Device Dimension](#device-dimension)

[Viewport](#viewport)

[References](#references)

近日公司接了一个html5 canvas的游戏开发，游戏本身的流程逻辑并不复杂，但canvas的游戏开发也是本人第一次做，期间也遇到了不少的坑，有些是自己挖的， 有些是android的坑，不过最终一切都往好的方面发展。之前一直没写过blog心得之类的，请各位看官手留情.

刚开始开发这个游戏时，我是一股脑得把所有的元素动画效果都放在一个canvas的完成，运行的效果呢也是挺不错的在电脑上，基本上帧数也都在60以上。此时的我也是对这个刚出生的婴儿挺喜欢的。但是好景不长，等把这个游戏在移动设备上测试的时候，才发现移动设备性能和电脑性能上的差距。ios设备上还好, 挺佩服苹果的，虽然硬件上比不上一些android的手机, 但是游戏性能上完爆大量的android的设备，有些android的设备上的自带浏览器对canvas的支持有多差我都不好意思说了. 游戏如果在电脑上性能再好，在移动设备上不行也是白搭，而且这个时候马上就要出测试版了，主要平台还是移动设备，所以这个时候也就开始了我的性能优化之路。

#SetTimeout or RequestAnimationFrame

一直以来，对动画的帧数的控制，我们一般都是使用`setTimeout` 或者 `setInterval`.

    var fps = 1000 / 60;

    function updateUI(){
      // do something here
      setTimeout(updateUI , fps);
    }

    updateUI();

现在大部分浏览器已经支持了一个新的方法：`requestAnimationFrame`. 使用`requestAnimationFrame`有多个好处：

- 浏览器可以对它进行优化，进而动画会更加流畅
- tab处于非活动时，动画会停止，进而cpu可以空闲下来
- 可以减少耗电, 这对于移动设备来说是相当关键的

`requestAnimationFrame` 的使用和 `setTimeout`基本上没有什么区别：

    function updateUI(){
      // do something here
      requestAnimationFrame(updateUI);
    }

    requestAnimationFrame(updateUI);

##Control your Game FPS

看了以上代码，你也许想说，并不是所有的动画都是要最高帧数的，我些动画只需要`20fps`就够了， 高帧数反而不是我想要的，那么如何对以上代码进行帧数控制呢。

答案是： 我们自己写一个控制帧数的方法.

    var Game = function(){
      ....
      //记录上一帧的timestamp
      this.then = Date.now();
      ....
    }

    Game.prototype.fpsGuard = function(fps, callback){
      var current   = Date.now();

      //每帧所需用时
      var interval  = 1000/fps;

      //当前帧与上一帧之前的时间差
      var delta     = current - this.then;

      if(delta > interval){
        then = current - (delta % interval);
        callback();
      }
    }

    Game.prototype.play = function(){
      this.fpsGuard(20, function(){
        ....
        doSomthing();
        ....
      }.bind(this));

      requestAnimationFrame(play);
    }

`fpsGuard` 就是我们用来控制帧数的方法. `fpsGuard` 接受2个参数：你所要控制的 `fps` 和 一个 `callback`. `fpsGuard` 每次调用的时候都会进行一次判断，看是否当前后帧的时间差小于等于当前 `fps` 所需用时. 如果是，则不做任何事情， 反之, 如果时间差大于当前`fps`所需用时, 刚调用回调函数, 也就是 `play` 方法, 并更新 `then` 的时间戳.

#Canvas Size

控制canvas的大小, canvas尺寸越大，相应的所需的资源也越多，意味着需要消耗更多的GPU资源和内存，同时也会增加耗电. 选择一个合适的尺寸,  一般移动设备上不应该超赤720P， 同时也要确保图片资源的分辨率也应该与canvas的尺寸保持一致，也就是说图片无需在cavnas上进行缩放, 也就意味着减少了对canvas的操作.

#Multiple Canvases

一般游戏可以分离出多个场景, 都会有一个前景和后景. 如果背景是一幅比较大的图片，并且，该背景在游戏开始后也无变化，此时我们就可以把背景和游戏动画分离开来.

    <style>
      #background, #foreground {
        position: absolute;
        top: 0;
        left: 0;
      }

      #foreground {
        background-color: transparent;
        z-index: 1;
      }
    </style>

    <canvas id="background" width='320' height='548'></canvas>
    <canvas id="foreground" width='320' height='548'></canvas>

#Off-Screen Canvas

*Off-Screen Canvas* 指得是一个不在dom树中的cnavas元素。`off-screen canvas` 的使用一般是用于预渲 (pre-render)

    <body>
      <canvas id='on-screen' width='320' height='548'></canvas>
    </body>

`canvas#on-screen`是一个on-screen canvas. 而一个 `off-screen` canvas则是创建后并不将其添加到dom树上.

    <script>
      var offScreenCavnas = document.createElement('canvas');
    </script>

在使用 `off-screen canvas` 前，你的代码可能是这样的:

    var ctx = document.getElementById('on-screen').getContext('2d');

    var Game = function(ctx){this.ctx = ctx};

    Game.prototype.drawHat = function (){
      this.ctx.drawImage ...;
    }

    Game.prototype.drawBody = function(){
      this.ctx.drawImage ...;
      this.ctx.fillRect ...;
    }

    Game.prototype.updateCharacter = function(){
      this.drawHat();
      this.drawBody();
    }

    var game = new Game(ctx);
    game.updateCharacter();


而当你使用 `off-screen canvas` 后，以上的代码则变为:

    var ctx = document.getElementById('on-screen').getContext('2d');

    var Game = function(ctx){
      this.ctx = ctx;
      this.offSceen = document.createElement('canvas');
      this.offSceen.width = 320;
      this.offSceen.height = 548;
      this.cacheCTX = this.offSceen.getContext('2d');
    };

    Game.prototype.drawHat = function (){
      this.cacheCTX.drawImage ...;
    }

    Game.prototype.drawBody = function(){
      this.cacheCTX.drawImage ...;
      this.cacheCTX.fillRect ...;
    }

    Game.prototype.preRender = function(){
      this.drawHat();
      this.drawBody();
    }

    Game.prototype.updateCharacter = function(){
      this.preRender();
      this.ctx.drawImage(this.offSceen, 0, 0);
    }

    var game = new Game(ctx);
    game.updateCharacter();


##Device Dimension

一般，我们获取屏幕尺寸都是用 `window.innerWidth` 和 `window.innerHeight`. 在一般浏览器上这也是没有问题的，但是到了 `android webview`， `hybrid app`时，这样获取屏幕尺寸就会出问题了, 在一些 android 设备上，`window.innerWidth` 返回的值总是不正确，有时甚至为0， 但`window.outerWidth`又能正确返回. 为了解决这个问题我们只需设定一个延时就好。

    setTimeout(
      function(){
        window.gameWidth  = window.outerWidth   || window.innerWidth  || screen.width
        window.gameHeight = window.outerHeight  || window.innerHeight || screen.height
        ...
      }, 300);

##viewport

    <meta name="viewport" content="width=device-width, height=device-height, target-densitydpi=device-dpi, maximum-scale=1, minimum-scale=1, initial-scale=1"/>

如果你所做的网站并不是面对移动设备的，完全没有必要使用这个标签。 如果是，那么就很有必要使用这个标签。

##target-densitydpi到底要不要用

`target-densitydpi` 这个属性的使用，一开始我也是想了一会到底要不要用。每次打开chrome的debug工具时. 都会有一句提示 `target-densitydpi support is being deprecated`, chrome和safari都是不支持这个属性的。也就是那个时候我决定不使用它。在移动浏览器上进行测试的时候一切都是正常的，结果到了`android webview`的时候，UI 毁了, UI放大得不成样子. 因此考虑到安卓的适配问题，还是把这个属性加上吧，毕竟对ios也没有影响。

#结语

其实还有很多东西可以写，只是由于本人懒就先这样放着吧，到时哪天空了再补上一些.

#References

[http://www.programering.com/a/MjM5EjNwATM.htm](http://www.programering.com/a/MjM5EjNwATM.html)

[http://www.quirksmode.org/mobile/metaviewport/](http://www.quirksmode.org/mobile/metaviewport/)

[http://tgideas.qq.com/webplat/info/news_version3/804/808/811/m579/201412/298539.shtml](http://tgideas.qq.com/webplat/info/news_version3/804/808/811/m579/201412/298539.shtml)

[http://codetheory.in/controlling-the-frame-rate-with-requestanimationframe/](http://codetheory.in/controlling-the-frame-rate-with-requestanimationframe/)

[http://blog.csdn.net/rogeryi/article/details/10755119](http://blog.csdn.net/rogeryi/article/details/10755119)

[http://www.quirksmode.org/blog/archives/2010/04/a_pixel_is_not.html](http://www.quirksmode.org/blog/archives/2010/04/a_pixel_is_not.html)

[http://css-tricks.com/using-requestanimationframe/](http://css-tricks.com/using-requestanimationframe/)
