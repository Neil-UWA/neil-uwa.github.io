---
layout: post
title: "Gulp自动运行model相关测试"
categories: jekyll update
date: 2015-12-16 16:49:30
---

项目结构：

    |-- common
        | -- models
    	    |-- user.json
    	    |-- user.js
    |-- server/
    ...
    |-- test/
        | -- models
            |-- user.test.js  //user model对应测试
    |-- tasks
    ...

先来一个最简单的gulp 配置：

    import mocha from 'gulp-mocha';

    export function(gulp) {
      gulp.task('mocha', ()=>{
    	return gulp.src(['test/**/*.js'], {read: false})
    			.pipe(mocha({reporter: 'spec'}));
      })
      gulp.watch(['test/**/*.js'], ['mocha']);
    }

以上配置对于测试变动时自然没问题，但是我们希望当我们对model进行修改时，自动运行测试心确保对model的修改不破坏已有的功能与特性。因此，以上代码变更为：

    export function(gulp) {
      gulp.task('mocha', ()=>{
    	return gulp.src([
    	  'test/**/*.js',
    	  'common/modal/**/*.js'
    	], {read: false}).pipe(mocha({reporter: 'spec'})
      });
      gulp.watch(['test/**/*.js'], ['mocha']);
    }

做了如修改之后 ，当我们对model进行任何修改，测试都会自动运行。但是这个配置有一个不足之处。当测试用例不多时该配置没任何问题，也花不了多少时间，但是一旦随着项目变大，测试用例变多，对任一一个model的修改都会跑一遍所有的测试是不可接受的，除非你想在测试的时候喝一杯咖啡。

因此我们想再进一步优化测试，当model变动时，只让model相关的测试跑一遍.

    export function(gulp) {
      gulp.task('watch:models', ()=>{
         var watcher = gulp.watch([
          'common/**/*.js',
          'common/**/*.json',
          'test/**/*.test.js'
         ], ['']);

        watcher.on('change', (e)=>{
          var filename = e.path.substr(e.path.lastIndexOf('/')+1).split('.')[0];
          gulp.src(`test/**/${filename}.test.js`, {read: false})
          .pipe(mocha({reporter: 'spec'}));
        });

      })
    }

如此修改之后，任何对model相关的修改都会促发运行与其相关的测试。当然上面还只是一个比较简易的配置，如想再继续优化，刚可以使用gulp-filter， gulp-changed 等，及使用正则做符合你项目设计的测试配置。

