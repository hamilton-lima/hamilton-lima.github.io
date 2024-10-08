---
title: 'Exploring NestJS: A Personal Journey and Key Insights'
date: Mon, 07 Oct 2024 21:31:00 +0000
draft: false
tags: ['nestjs', 'express', 'typescript', 'api']
---

Time flies! It feels like just yesterday when [Qi Chen](https://www.linkedin.com/in/qi-chen-ca) introduced [NestJS](https://nestjs.com) as our backend technology at Mindbeacon back in 2019. It was a perfect match for Angular—similar coding styles, dependency injection, plenty of decorators, and loads of CLI options. Today, NestJS (not to be confused with its cousin, [Next.js](https://nextjs.org)) can do so much more: scheduling, support for RabbitMQ, Kafka, Redis, WebSockets, and the list goes on.

I might be a bit biased, having used it for the past five years, but sometimes it feels like I've not just used it—I've pushed it to its limits! 😅 Especially with all the extensions I built while working at Modern Health Canada.

While the documentation is very detailed and full of great examples, I want to highlight some key aspects that combine more than one topic from the NestJS documentation:

- **Custom Decorators**: Explore `createParamDecorator()`, which is very similar to how Python creates custom decorators. You have a function with the context for the current execution, and you can add the necessary behavior to it. For example, you can extract data from a JWT token and assign it to a variable so it can be consumed by all methods.

- **Swagger Generation**: It's just one annotation away, and with that, SDK code generation becomes incredibly useful. You get not only an SDK to call your APIs but also data structures from the server that are exposed to the client (as long as they're properly annotated). A word of advice when adding these annotations: include `operationId`, otherwise the code generation tool will need to guess the method names in the SDK. For example: `@ApiOperation({ operationId: 'createPerson' })`.

- **[swaggerStats](https://www.npmjs.com/package/swagger-stats)**: This is your bridge to Prometheus and an easy-to-consume stats user interface. Usually, a single line will enable it: `app.use(swaggerStats.getMiddleware({ swaggerSpec: document }))`, and voilà!

- **[@nestjs/terminus](https://www.npmjs.com/package/@nestjs/terminus)**: This package not only comes with an out-of-the-box way to check if the connection with the database is alive and well, but it's also easy to extend to check connectivity with Redis.

- **[ValidationPipes](https://docs.nestjs.com/techniques/validation#auto-validation)**: These can ensure that the content of requests follows specific rules and enable very common cases of data validation, like `@IsEmail()`, `@IsArray()`, `@IsNotEmpty()`, and so on. See the complete list [here](https://github.com/typestack/class-validator#validation-decorators).

There's so much more to explore! I recommend going through the list of techniques at [docs.nestjs.com](https://docs.nestjs.com/). Have fun!
