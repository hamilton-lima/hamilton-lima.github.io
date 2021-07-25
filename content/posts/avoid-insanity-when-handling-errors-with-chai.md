---
title: 'Avoid insanity when handling errors with Chai'
date: Sat, 25 Jul 2021 10:34:00 
draft: false
tags: ['testing', 'chai', 'typescript', 'smallflatworld']
---

![Vaas from Farcry3](/images/2021/insanity.png) 
If you ever played Farcry3 for sure you will remember the guy in the picture, if you never hear about him, Take a look on this video, [The definition of insanity - Far Cry 3](https://www.youtube.com/watch?v=itjmKlYjUak). 

While coding some unit tests for the SmallFlatWorld server, I got myself in a very interesting situation, where my old Java memories guided me to implement a test in a specific way, while some documentation was pointing in another direction... Well Vaas has a point, repeating the same thing over and over and expecting the same results it's kinda insane. Then I try multiple ways to do the same thing, let's take a look.

### What is this test about?

- Verify if a specific scenario will throw an exception
- Make a simple and readable test
- Write test that I can understand without a need for searching about it
- Minimize brainpower need to decypher the test so I can change without fear

## What are the options I tried

### The magician bind!
```
expect(this.storage.getStorage.bind(this.storage, "foo")).to.throw(
  "uuid dont exist in memory"
);
```

If you look at the documentation you will find that [bind](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_objects/Function/bind) will create a new function, with the `this` reference you passed as argument, plus other arguments you give to it.

In other words, a function in this case `getStorage` can be recycled to be used in other instance, so other `this` reference and be called by the `expect` function and then we can check for the the error using `to.throw`.

As I don't `bind` most of the time, this solution seems very fuzzy, it works but I would need a quick read to recap, and answer my expected reaction to this code in couple months: "Why on earth am I using `bind` here?"

### Anonymous functions FTW!

```
const self = this;
expect(function () {
  self.storage.getStorage("foo");
}).to.throw("uuid dont exist in memory");

expect(() => {
  this.storage.getStorage("foo");
}).to.throw("uuid dont exist in memory");
```

Both solutions here rely on creating an anonymous function, and letting `expect` to execute them. 

But there is a gotcha, there is always a gotcha, when you create a function using `function ()` syntax, and you code is running on stric mode, that function `this` will be `undefined`! for this reason you can see a not so cool workaround, as know as patch and for me a fine example of [Go horse programming](https://gist.github.com/banaslee/4147370), the `const self = this`. that provides the existing `this` reference to the function.

The second version, using the arrow syntax, as defined here [Arrow functions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions) does NOT have it's own `this` reference, so it will rely on the existing one, and that would work just fine.

Sending functions as parameters sometimes feels like blackmagic, I know how it works but requires some brainpower to process it, and for me tests should not be a place to exercise theses powers.

![Brain you have no power here](/images/2021/you-have-no-power-here.png) 

### The verbose, but clear
```
try {
  this.storage.getStorage("foo");
} catch (error) {
  expect(error.message).to.be.equal("uuid dont exist in memory");
}
```

This is the verbose version, but very clear what is going on, the code starts with `try {`, even if you don't know the language, or if you are very tired, or watching a video while reading the code, you will be able to understand that something is expected to go wrong in the next lines.

The piece of code that is under test is called the same way it will be used in other parts of the code, so again no magic here, no mental transformation, no extra variables to hunt you with the question, will this impact the test? just a function call `this.storage.getStorage("foo");`

Then at the end a `catch` with a `expect` in it, saying out loud: "This test expects the code above to fail, and to fail with this message".

For me this is the winner :)

### References 

- [Chai to.throw](https://www.chaijs.com/api/bdd/#method_throw)
- [smallflatworld test code reference]()