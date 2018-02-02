Title: How to improve Javascript coding
Date: 2015-03-11 22:47
Author: Surya
Category: Programming
Slug: how-to-improve-javascript-coding
Status: published

**Advantages**

1.  Modern browsers are way faster to process JS.
2.  Trend – clients are doing what servers used to do
3.  Server is used only to generate data like json
4.  Client is used only to generate/render html
5.  Speed/efficient (no redundant data transfer between server client)

**Disadvantage**

1.  No compile time error checking
2.  Once a project becomes big, it is hard to upgrade
    libraries/framework  
    without breaking something (at runtime)
3.  Not properly organized (global methods, big script file)

**Solution**

1.  Enclose functions in namespace
2.  Modularize code (this way big file can be split)
3.  “use strict”
    ([reason](http://stackoverflow.com/questions/1335851/what-does-use-strict-do-in-javascript-and-what-is-the-reasoning-behind-it "why-use-strict"))
4.  Write unit test (to solve compile time checking)

here is an example - 

    :::javascript
    (function (nsr, $, undefined) {
         "use strict"
         nsr.Module1 = nsr.Module1 || {};
         nsr.Module1.variable1 = 'test';
         nsr.Module1.function1 = function(){}
    } (window.nsr = window.nsr || {}, jQuery));
