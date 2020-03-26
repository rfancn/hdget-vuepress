---
date: '2017-04-13 10:21:06'
draft: false
lastmod: '2017-11-25 20:16:11'
categories:
- 博客
tags:
- program
- web
title: How to get just the responsive grid from Bootstrap 3?
---

Recently, I am plan to write some html page use weui css, but it seems it lack good grid system, as we all know bootstrap3 grid system is very easy to use, I am considering to get responsive grid-only css stuff from bootstrap3. In fact, it does already exist a opensource project: [bootstrap-grid-only](https://github.com/zirafa/bootstrap-grid-only/)

But it seem author didn't update it for long time, the customized less file which used to generate grid-only css contains much copy-&-paste stuff. To keep things more simple and readable, I tried to create a customized less file by using official less files:

1. Get bootstrap source file from github: https://github.com/twbs/bootstrap/archive/master.zip
2. Extract and go to 'bootstrap-master/less' directory
3. Copy utilities.less file to 'utilities-grid.less'
4. Remove all stuff in 'utilities-grid.less' file, and only leaves(if fact, only clearfix needed, but for convenience purpose, leave pull* and hide*):

        // Floats
        // -------------------------
        
        .clearfix {
          .clearfix();
        }
        .pull-right {
          float: right !important;
        }
        .pull-left {
          float: left !important;
        }
        
        
        // Toggling content
        // -------------------------
        
        // Note: Deprecated .hide in favor of .hidden or .sr-only (as appropriate) in v3.0.1
        .hide {
          display: none !important;
        }
        .show {
          display: block !important;
        }
        .invisible {
          visibility: hidden;
        }
        
        // Hide from screenreaders and browsers
        //
        // Credit: HTML5 Boilerplate
        
        .hidden {
          display: none !important;
        }

5. Copy scaffolding.less to 'scaffolding-grid.less' and only leaves:

        // Reset the box-sizing
        //
        // Heads up! This reset may cause conflicts with some third-party widgets.
        // For recommendations on resolving such conflicts, see
        // http://getbootstrap.com/getting-started/#third-box-sizing
        * {
          .box-sizing(border-box);
        }
        *:before,
        *:after {
          .box-sizing(border-box);
        }

6. Copy bootstrap.less file to 'bootstrap-grid.less'
7. Remove all stuff in 'bootstrap-grid.less', and only leaves:
    
        @import "variables.less";
        
        @import "mixins/clearfix.less";
        @import "utilities-grid.less";
        
        @import "mixins/grid-framework.less";
        @import "mixins/grid.less";
        @import "grid.less";
        
        @import "mixins/vendor-prefixes.less";
        @import "scaffolding-grid.less";
        
        @import "mixins/responsive-visibility.less";
        @import "responsive-utilities.less";

7. Generate css file using lessc

   $ lessc --verbose bootstrap-grid.less bootstrap-grid.css