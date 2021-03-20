---
title: 'beepify.io - start'
date: Sun, 16 Oct 2016 12:51:26 +0000
draft: false
tags: ['beepify.io', 'documentation', 'poll', 'selfhackaton', 'slack', 'start', 'swagger', 'webservice']
---

This will be one selfhackton project, to explore [play framework](https://playframework.com/) as server side implementation and [Ionic framework](http://ionicframework.com/) at the client side What is the main value of beepify.io?

> Offer product information at your fingertips.

Beepify.io will allow the customer to receive product information by scanning the product bar code.

### The start

As I am planning to participate on the next Vanhackton, I invited other participants to start a project to warmup on some technologies:

*   [Jonathan Barros - https://jonathancbarros.github.io](https://jonathancbarros.github.io)
*   [Cristian Echeverria - https://crisecheverria.github.io](https://crisecheverria.github.io)
*   [Andre Luiz - https://andreluizreis.github.io](https://andreluizreis.github.io/)

We decided together the overall details of the application but decided that each one should build his own implementation. First we had several ideas of what information to send to the customer about the product, the initial list was big:

*   prices
*   location
*   nutricional information
*   similar products
*   feedback from users
*   reviews in stars
*   talk to the company
*   data from the user
*   pictures
*   special info from the manufactor
*   prices from stores
*   product details details from the company
*   product details from the customers

but after some voting, we managed to reduce the list to only five items:

*   compare prices
*   add user reviews
*   create product lists
*   chat with customer service
*   search in stores nearby

After doing a quick public opinion poll at the vanhackton slack channel that has around 600 users, we got the feedback we need: compare prices and search in stores nearby were the public choice. Here are the poll results ![beepify-poll](/images/2016/10/beepify-poll.png) We use the following command to create the poll:
```
/poll "We the beepify team are warming up for the hackton, so we decided to create an application that will give information about products. What feature you choose to be the first one ?" "Compare prices" "Add user reviews" "Create product lists" "Chat with customer service" "Search in stores nearby"
```

With that results we decided to merge the compare prices with the search in stores nearby in one feature.

### The application flow

These are the steps that the application should follow: 1 - When the application opens it should be ready to scan the bar code 2 - As soon it detects a bar code, it should beep, blink the screen and redirect to the results screen 3 - The results screen shows a map with products and prices, pointing at the map the stores. The following are some suggestion of UI implementation:

*   a - 50% with map + 50% with the list of products and prices
*   b - 100% map + button or action to show the list of products
*   c - other option

The data structures and server side services is documented using [http://swagger.io](http://swagger.io), in order to see the documentation file go to the [Swagger online editor](http://editor.swagger.io/) and paste the content of this file: [beepify-io-yaml](/images/2016/10/beepify.io_.yaml_.txt) The result will be a nice and organized documentation of the data structures and services parameters like this ![2016-10-16_0944](/images/2016/10/2016-10-16_0944.png)