---
layout: post
title: "Strongly Typed Properties for JavaScript"
description: ""
category: 
tags: [javascript, strong typing]
---
{% include JB/setup %}

While coding in JavaScript I've often wished I could enforce the type of some variables.
The typical case where I've felt the need for that is when some mathematical operation or comparision is
done on a non-number, yielding wierd results but not throwing an error.

While tinkering with a hobby project (an HTML5 game) I repeatedly run into the case where
a `NaN` was found where a number was expected. `NaN` is usually the result of a mathematical operation gone wrong,
is of type number, and is hard to reason about.

Consider the following:

    sprite.y = sprite.y + sprite.velocityY;

If sprite.velocityY is undefined or `null`, sprite.y will be `NaN` and the JavaScript interpreter won't complain. I just can't
understand why the sprite on screen isn't moving.

This is the type of error I encountered. At first I tried to track such bugs down by placing
checks everywhere. However, it's just after-the-facts-checking and it didn't tell me where the value was corrupted.

### The Solution

Enter [Object.defineProperty](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty). Most browsers support this feature today. When constructing an object, you can create a property
with a getter and setter that acts just like a normal variable, but you have the opportunity to inspect the value. If you wanna apply this to a class, apply it to the prototype.
Consider the example with the sprite:

    // Class definition

    function Sprite() {
      // Constructor code ...
    };
  
    Object.defineProperty(Sprite.prototype, "y", {
      set: function(val) {
        if (typeof val !== "number" || isNaN(val)) {
          throw "Sprite.y must be a valid number and not NaN";
        }
        this.__y = val;
      },
      get: function() {
        return this.__y;
      }
    
    })
    
    Sprite.prototype.y = 0;
    Sprite.prototype.x = 0;
    
    // Instantiate
  
    var sprite = new Sprite();
    sprite.x = "Hello, world!"; // Passed without complaints
    sprite.y = sprite.somethingUndefined / 10; // Exception is thrown

...and suddenly you have an object with a strongly typed property.

Wrapping this in a function would look something like:

    function typedProp(obj, key, type) {
      if (!Object.defineProperty) { // Make older browsers not crash
        return;
      }
      Object.defineProperty(obj, key, {
        set: function(val) {
          if (typeof val !== type || (type === "number" && isNaN(val))) {
            throw "Property '" + key + "' should be of type '" + type + "', value is: '" + val + "' of type '" + typeof val + "'";
          }
          this["__" + key] = val;
        },
        get: function() {
          return this["__" + key];
        }
      });
    }
    
    
    // ... and augment the Sprite prototype:
    
    typedProp(Sprite.prototype, "name", "string");
    
    sprite.name = 31337;


Since you might wanna avoid running this in a production environment, you could just have typedProp be an empty function in your release build.
At least you should check for the presence of `Object.defineProperty`.

### Some Syntactic Sugar

In my own class library, I implemented the functionality to be used as such:

    CB.Class("Sprite", {
      "x:number": 0,
      "y:number": 0,
      "name:string": "defaultname",
      
      initialize: function() {
      }
      // More code goes here
    });
  
What happens here is that I added some syntactic sugar that lets you define properties in ActionScript-like-style.
Yes, you can have colons in property names. The part after the colon is stripped off and used as the type.
Use my library if you want to: [git@github.com:zufallsgenerator/cbclass.git](https://github.com/zufallsgenerator/cbclass), but you probably have itchy finger to build your own.

Happy coding!


_Edit: strictly typed -> strongly typed_
_Edit: fixed typo_
