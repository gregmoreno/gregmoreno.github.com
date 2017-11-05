---
layout: post
title: 5-bullet Sunday morning"
date: 2017-11-04 17:50:57 -0700
comments: true
categories: 
---

*I’m currently reading [Principles by Ray Dalio](https://www.amazon.ca/Principles-Life-Work-Ray-Dalio/dp/1501124021/ref=pd_bxgy_14_img_2?_encoding=UTF8&psc=1&refRID=BMW026RN9BH7JBJKAFXP)*

When your name is Ray Dalio, you dont need a thought-provoking title nor a fancy cover to have a bestseller. It is a thick book and with a hardbound black cover, it can pass as the Bible. Just like the Bible, this is not a book you read in one sitting nor something to pass the time. I had to pause, think about what he said, question it, and see if I can apply it to my life.

Here is Ray Dalio’s popular [TED talk](https://www.youtube.com/watch?v=HXbsVbFAczg).

*Bitcoin is so hot I need to mention it*

A friend messaged me complaining how complicated and tedious it is to trade Bitcoin and other cryptocurrencies. Worse, he is just halfway to all the things he need to do. We haven’t fully discussed what a wallet is for, why a piece of paper can be so important in the Internet age, and why he needs a water-tight fire-proof vault :) He made the leap even though he still doesn’t fully understand what all these currencies are and I can only imagine how these steps can feel like just a waste of time.

He is not alone and this is good thing.

Yes, I am channeling my inner Stoicism here so here me out. Massive financial gains happen when you got in an opportunity before the general public does. The Bitcoin ecosystem right now is not much different from the early days of the Internet — with all the possibilities, confusion, as well as danger of [another bubble](https://medium.com/@dennyk/why-and-how-the-cryptobubble-will-burst-de9bc7fc5332).

If you feel the pain, look at it as your ticket to an exciting game. Let us enjoy the game and I wish you good luck.

*On watching Cirque du Soleil*

I first heard of Cirque du Soleil about 4 years ago. Whenever they are in Vancouver, they would have a big tent setup in the parking lot near a train station. Around this time, I would see that tent on way to and from my work. It is like one mysterious event. You don’t see much from the outside but people are lining up and you hear conversations in the train on how breathtaking the show is.

When I heard they are coming again, we made sure we will not let it pass this time, damn the cost. I am glad we did not. My kids love it. I love it. Seeing people fly and do stunts in a 3D theatre is fun though we know it is all computer magic and stunt doubles. But watching people somersault in the air with only a rope on one arm is way different fun. You can actually feel the danger with it.

I am grateful I get to watch this with my family.

*This thing in JavaScript*

Finished some modules from Wes Bos JavaScript course, so why not apply it right away. So I did and I thought this was no a brainer.

    $('#filters').on('change', () => {
      console.log($(this).val());
    });

Except that it is not and I paid some scarce hair for it. The right solution is to use regular function.

    $('#filters').on('change', function () {
      console.log($(this).val());
    });

Ah the binding. When using arrow function, `this` is not bound to the `on` scope. Instead, it just inherited whatever the parent scope is. Lesson learned.

Now I get it. So I thought.

    $('#filters').on('change', function () {
      console.log($(this).val());
      setTimeout(function () {
        console.log($(this).val());
      });
    });

Damnit, same error again. Because inside the function passed to setTimeout, `this` is bound to `setTimeout`. Hence, the same error. To fix it, just use arrow function.

    $('#filters').on('change', function () {
      console.log($(this).val());
      setTimeout(() => {
        console.log($(this).val());
      });
    });

Confusing, eh? Just remember to go back to the binding rules for the arrow function and now it makes sense because we want to have this refer to the parent scope.

*Quote for you*

"A side effect of doing challenging work is that you’re pulled by excitement and pushed by confusion at the same time."
- [James Clear](https://jamesclear.com/successful-people-start-before-they-feel-ready?__s=rg5u4ofe9tzwuyfixegg)

This post also appeared at [Medium](https://medium.com/@gregmoreno/5-bullet-sunday-morning-1524f4db811).
