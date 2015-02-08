---
layout: post
title: "HTML5 Canvas游戏开发心得"
categories: jekyll update
tags: canvas, performance optimization
---

近日公司接了一个html5 canvas的游戏开发，游戏本身的流程逻辑并不复杂，但canvas的游戏开发也是本人第一次做，期间也遇到了不少的坑，有些是自己挖的， 有些是android的坑，不过最终一切都往好的方面发展。之前一直没写过blog心得之类的，请各位看官手留情.

刚开始开发这个游戏时，我是一股脑得把所有的元素动画效果都放在一个canvas的完成，运行的效果呢也是挺不错的在电脑上，基本上帧数也都在60以上。此时的我也是对这个刚出生的婴儿挺喜欢的。但是好景不长，等把这个游戏在移动设备上测试的时候，才发现移动设备性能和电脑性能上的差距。ios设备上还好, 挺佩服苹果的，虽然硬件上比不上一些android的手机, 但是游戏性能上完爆大量的android的设备，有些android的设备上的自带浏览器对canvas的支持有多差我都不好意思说了. 游戏如果在电脑上性能再好，在移动设备上不行也是白搭，而且这个时候马上就要出测试版了，主要平台还是移动设备，所以这个时候也就开始了我的性能优化之路。

#SetTimeout or RequestAnimationFrame

##How to control your game fps

#Canvas Size

#Composite Canvas


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

    Game.prototype = {
      drawHat: function (){
        this.ctx.drawImage ...;
      },

      drawBody: function(){
        this.ctx.drawImage ...;
        this.ctx.fillRect ...;
      },

      updateCharacter: function(){
        this.drawHat();
        this.drawBody();
      }
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

    Game.prototype = {
      drawHat: function (){
        this.cacheCTX.drawImage ...;
      },

      drawBody: function(){
        this.cacheCTX.drawImage ...;
        this.cacheCTX.fillRect ...;
      },

      preRender: function(){
        this.drawHat();
        this.drawBody();
      },

      updateCharacter: function(){
        this.preRender();
        this.ctx.drawImage(this.offSceen, 0, 0);
      }
    }

    var game = new Game(ctx);
    game.updateCharacter();


##获取设备尺寸

##viewport

    <meta name="viewport" content="width=device-width, height=device-height, target-densitydpi=device-dpi, maximum-scale=1, minimum-scale=1, initial-scale=1"/>

如果你所做的网站并不是面对移动设备的，完全没有必要使用这个标签。 如果是，那么就很有必要使用这个标签。有些人就会问：‘为什么呢？‘。

##target-densitydpi到底要不要用

`target-densitydpi` 这个属性的使用，一开始我也是想要一会到底要不要用。每次打开chrome的debug工具时. 都会有一句提示 `target-densitydpi support is being deprecated`, chrome和safari都是不支持这个属性的。也就是那个时候我决定不使用它。在移动浏览器上进行测试的时候一切都是正常的，结果到了`android webview`的时候，UI 毁了, UI放大得不成样子。

有一点需要注意的是，设备象素 (*device pixel*) 和 CSS 象素 (*css pixel*) 是不一样的。 对于网站开发人员来说，我们需要的是 `css pixel`

In Android browser and early versions of Chrome for Android, developers could use target-densitydpi=device-dpi viewport value to force the browser to make a CSS pixel the same size as a device pixel,

#References
[http://www.quirksmode.org/mobile/metaviewport/](http://www.quirksmode.org/mobile/metaviewport/)
[http://tgideas.qq.com/webplat/info/news_version3/804/808/811/m579/201412/298539.shtml](http://tgideas.qq.com/webplat/info/news_version3/804/808/811/m579/201412/298539.shtml)
[http://codetheory.in/controlling-the-frame-rate-with-requestanimationframe/](http://codetheory.in/controlling-the-frame-rate-with-requestanimationframe/)
[http://blog.csdn.net/rogeryi/article/details/10755119](http://blog.csdn.net/rogeryi/article/details/10755119)
