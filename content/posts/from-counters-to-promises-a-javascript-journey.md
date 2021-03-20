---
title: 'From counters to promises a javascript journey'
date: Tue, 28 Feb 2017 13:21:21 +0000
draft: false
tags: ['javascript', 'mask', 'maskeditor', 'promise', 'robolucha']
---

> Beauty always promises, but never gives anything. [Simone Weil](https://en.wikipedia.org/wiki/Simone_Weil)

Unlike beauty, javascript promises are faithful, and will be fulfilled... unless someone rejects it :) If you ever need to load images using javascript or did anything that needs to wait until something is done, you probably used a promise, maybe even without noticing. I see Promises as a nice way to simplify the [callback hell](http://callbackhell.com), but don't be fooled, the callbacks are still there, they are only organized in a more simpler way. Let´s take a look at a simple example that loads one image. consider that "foo" variable is a canvas where we draw something in memory and we want to show that as an image.
```
var promise = new Promise(function(resolve, reject) {   
    var image = new Image();
    image.onload = function() {
        resolve(image);
    }
    image.src = foo.toDataURL("image/png");
});
...
promise.then(function(data){
    myGreatImage = data;
});
```

The code above says: "When I'm done loading the image you can have it". the resolve function from the promise is calling the then implementation, don't tell anybody but there is a callback here... :) but with a nice name and a very readable structure.

Promises on robolucha masks
---------------------------

During this weekend I finally add promises to the mask implementation of robolucha. The original implementation was handling image loading by counting the number of images loaded and with a ready method, but after the image loading the tint of the images and the mask building itself wasn't controlling the asynchronous nature of the operation, as a result of it, in Chrome it was working but at Firefox wasn't. The table below describes the features from the robolucha mask builder and the asynchronous challenges from each one.

|Feature|Implementation|Challenge|
|-|-|-|
|Base image loading|Add onload to each image and control the number of images loaded to track if the set of images are ready to be used|Wait for all the images to load|
|Mask building|Draw tinted images on a canvas and return the base64 image data as src of the resulting image|Wait for all layers to tint and after it wait for the mask to draw|
|Tint an image layer|Paint the black layer pixels with the chosen color|Wait for the layer to tint|

The great challenge was at the mask building where the code needed to wait for several promises to resolve and after that return a promise, so this is the implementation of the mask build.

```
this.build = function(width, height, data) {
	
	var imagePromises = new Array();
	
	for (var i = 0; i < data.length; i++) {
	    var layer = this.layers\[data\[i\].name\]; 
	    if (layer) {
		var rgb = this.hexToRgb(data\[i\].color);
		imagePromises.push(this.tint(this.findImageByName(data\[i\].name, data\[i\].element), 
                     rgb.red, rgb.green,rgb.blue));
	    }
	}
	
	var resultPromise = new Promise(function(resolve, reject) {
	    Promise.all(imagePromises).then(function(images) {
		var resultCanvas = document.createElement("canvas");
		resultCanvas.width = width;
		resultCanvas.height = height;
		var ctx = resultCanvas.getContext('2d');
		ctx.imageSmoothingEnabled = true;
		
		for (var i = 0; i < data.length; i++) {
		    ctx.drawImage(images\[i\], 0, 0, width, height);
		}
		
		// return the base64 string of the image
		resolve(resultCanvas.toDataURL("image/png"));
	    });
	});
	
	return resultPromise;
    }

```

Let's talk about some important steps on this code:
```
imagePromises.push(this.tint(...
```

The "tint" function returns a promise that will be resolved when the image painting is done. We add to the list of promises that we are waiting for.
```
var resultPromise = new Promise(function(resolve, reject) {
	    Promise.all(imagePromises).then(function(images) {...
```

The "Promise.all" function allow us to wait for a list of promises and do something at the "then" function, however, the parameter received at the function is the list of data each promise sends from "resolve" function. ![](/images/2017/02/promises-image-to-promisse.all_.jpg) With "Promise.all" I was able to solve one problem that was wait for several other promises, but now I had another one how to transform the result list in one result that should be a "promise" result? The solution I found was to create a new promise with "Promise.all" inside so the resolve function would be available.

```
resolve(resultCanvas.toDataURL("image/png"));
```

This resolve function is from the new promise, so when the "then" implementation of the "Promise.all" finished it will resolve this new promise sending the image as base64. If you know any other way around this, please let me know :)