---
title: 'should price be an attribute ?'
date: Wed, 19 Oct 2016 22:32:59 +0000
draft: false
tags: ['attribute', 'beepify.io', 'data model', 'entity', 'price', 'selfhackaton']
---

Just after I post about the data structure and server side implementation of beepify.io, I got a question about the Price entity. One very experienced guy on SAP technologies said:

> I never saw Price as an entity, allways saw as attribute, why you made Price an Entity?

And support his question he also send some example structure. ![should-price-be-an-attribute](/images/2016/10/should-price-be-an-attribute.png) My first was, I had no idea why the model has price as entity :) but after some thinking, I just realized that I model in the way I tought about the business. So the Price information for this business is more relevant than the store itself. We could change the model to be: Product > Store > Price, but at the end it makes more sense to have: Product > Price > Store. Of course there is the argument that we could have a price without a store and add the store in the future, but this argument could be overcome by one separated table for the new prices, that could be batch processed to add the stores and then be part of the main table. As conclusion I beleive that modeling should follow the natural thinking of the business, so it should make sense for whom will use a lot, and sometimes both solutions are good. Saying that I keep: **Product > Price > Store** ![beepify-database](/images/2016/10/beepify-database.png) Thanks for the feedback: [Ramon Santos](https://br.linkedin.com/in/ramon-gomes-santos-0768181a)!