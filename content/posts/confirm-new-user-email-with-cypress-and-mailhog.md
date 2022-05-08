---
title: 'Confirm new user email with Cypress and MailHog'
date: Sun, 08 May 2022 09:09:09 +0000
draft: false
tags: ['testing', 'email', 'cypress', 'mailhog','automation']
---

I really like [Cypress](https://cypress.io), The tool is interactive, very developer friendly even if you don't plan to have a fully automated test suite, it really can save time automating tasks like, fill some forms, repeat some tasks, reproduce bugs, create new users... create new USERS... oh wait CREATE NEW USERS, how on earth I will validate the user email?

![Gru showing steps of user creation with cypress](/images/2022/user-creation-automation-steps-cypress.jpg) 

## So many options
There are some posssibilities
- Pay some email API services, oh GMAIL why you don't have it?
- Navigate to mailinator webpage, parse the HTML and find the email, I need internet for this.
- Pay mailinator API, did that in the past, works just fine, but you know... money.

Then some revelation come, not really just research on options, why not use a local email server then build an API to get the email contents? The first one that came to mind is [James](https://james.apache.org/) very stable email server, save data in the database, so it would be super easy to extract the messages, but why not an existing solution that is email server AND already have the API?

Then I found [MAILHOG](https://github.com/mailhog/MailHog)! Email server with API to fetch the data, oh this is good, very good :) 

## How it works?

So let's see the entire process, with some quick notes
- Start email server, docker-compose is your friend, this is the image: `mailhog/mailhog:v1.0.1` and these are the ports: 1025 and 8025
- we delete all messages from the local email server, call DELETE using `/api/v1/messages`
- visit the user creation page, `cy.visit()`
- enter the data for the new user, generate some fake date with `@faker-js/faker` then use multiple `cy.get().type()` 
- submit the form, `cy.get().click()`
- confirm that the API finished with success, `cy.intercept()` and `cy.wait()`
- the API will send and email to our mailhog local server, included a environment variable use [nodemailer](https://nodemailer.com) to send the messages to the test server instead of [mailgun](https://www.mailgun.com)
- get email received to the created user, call GET `/api/v1/messages` then `/api/v1/messages/{id}`
- find the confirmation link in the email, decode from `quotedPrintable` then parse the HTML using [cherio](https://cheerio.js.org)
- visit the link, `cy.visit()`
- login with the new account

## Show me the code 

Docker compose for the the email 
```
  email:
    container_name: email
    image: mailhog/mailhog:v1.0.1    
    ports:
      - 1025:1025
      - 8025:8025
    logging:
      driver: "json-file"
      options:
        max-size: "200k"
        max-file: "10"
```

Delete all the emails 
```
cy.task('deleteAllEmails', email).then(response => {
  cy.log(`All emails deleted from the mail box ${response}`);
})

// code in the plugin
...
const deleteAllEmails = async () => {
  const { data } = await axios.delete(`${MAILHOG_HOST}/api/v1/messages`);
  return data;
}
```

Visiting the signup page 
```
  beforeEach(() => {
    cy.visit(`${host}/signup/newuser`);
  })
```

Generating fake data and entering in the fields, remember to add the `id` to the input fields, there is always to option to use the [recommended cypress strategy](https://docs.cypress.io/guides/references/best-practices#Selecting-Elements), `cy.get('[data-cy=submit]').click()` but I prefer the syntax using the id `cy.get('#submit]').click()` as most of the time with the new frameworks we pretty much don't change component id, then this field is less volatile, and the syntax seems more readable.
```
  let email;
  let password;
  let firstName;
  let lastName;

  before(() => {
    password = faker.internet.password();
    firstName = faker.name.findName();
    lastName = faker.name.findName();
    email = faker.internet.email(firstName, lastName, 'host.docker.internal');
  })

... 
    cy.get('#firstName').type(firstName);
    cy.get('#lastName').type(lastName);
    cy.get('#email').type(email);
    cy.get('#password').type(password);
```

Submit the form and check if the API finished without problems
```
    cy.intercept('POST', '**/user/create*').as('CREATE')
    cy.get('#submit').click();
    cy.wait('@CREATE').its('response.statusCode').should('be.oneOf', [201])
```

Nodemailer sending email to local server WITHOUT authentication, note that it uses the same data structure as mailgun
```
  async sendEmail(data: MailgunData): Promise<string> {
    try {
      this.logger.log(`Send email local ${data.subject} TO: ${data.to}`);

      const transporter = nodemailer.createTransport({
        port: 1025,
        host: 'host.docker.internal',
        secure: false,
      });

      const result = await transporter.sendMail(data);
      this.logger.log(`Email sent ${JSON.stringify(result)}`);
 
      return result;
    } catch (error) {
      this.logger.error('Error when sending the notification', error);
    }
  }
...
export class MailgunData {
  subject: string;
  from: string;
  to: string;
  text: string;
  html: string;
}  
```

Getting the email messages
```
  const { data } = await axios.get(`${MAILHOG_HOST}/api/v1/messages`);
  ...
  // do some data massage to get the email ID, see the plugin full code at the end of this.
  ... 
  const { data } = await axios.get(`${MAILHOG_HOST}/api/v1/messages/${id}`);
  result = data;
```

DECODE the [quotedPrintable](https://www.npmjs.com/package/quoted-printable) then parse the HTML using [cherio](https://cheerio.js.org), OMG this one took some time, read some more here [Quoted Printable](https://en.wikipedia.org/wiki/Quoted-printable), in summary if you see `=3D` all over the HTML message there is a good chance you need to decode it :) 

After the decoding there was the challenge of parsing the message HTML, cypress offers this but not in the plugin code, so then I used [cherio](https://cheerio.js.org), my new friend, to parse the HTML, this is such a nicely done piece of technology, simple and very resourcefull. it works just as expected, feed in the HTML, get a parsed object, do queries to find elements, receive a LIST get the one you want and boom retrieve some attributes.

See the code, it speaks for it self
```
const getLinkFromEmail = (email, tag) => {
  console.log(`getLinkFromEmail ${email} ${tag}`);
  const body = utf8.decode(quotedPrintable.decode(email.Content.Body));
  const html = cheerio.load(body);
  const links = html(tag).find('a');
  const link = links[0];
  const result = link.attribs.href;
  return result;
}
```

BONUS - create a plugin for cypress

Add this to your `plugins/index.js`
```
/// <reference types="cypress" />
const mailHogAPI = require('./mail-hog.api')

/**
 * @type {Cypress.PluginConfig}
 */
module.exports = (on, config) => {
  on("task", {
    async "getEmail"(to) {
      return await mailHogAPI.getEmail(to);
    },
    async "deleteAllEmails"() {
      return await mailHogAPI.deleteAllEmails();
    },
    async "getLinkFromEmail"({ email, tag }) {
      return mailHogAPI.getLinkFromEmail(email, tag);
    }
  })
};
```

And the plugin file itself `plugins/mail-hog.api.js`
```
const axios = require("axios");
const cheerio = require('cheerio');
var quotedPrintable = require('quoted-printable');
var utf8 = require('utf8');

const MAILHOG_HOST = process.env.MAILHOG_HOST || 'http://host.docker.internal:8025';
// See API documentation here https://github.com/mailhog/MailHog/blob/master/docs/APIv1.md

const getEmail = async (to) => {
  if (!to) {
    return;
  } else {
    to = to.toLowerCase();
  }

  const { data } = await axios.get(`${MAILHOG_HOST}/api/v1/messages`);

  let id = null;
  let result = null;

  // search path []{ID,Raw:[{To:[string]}]}
  if (data) {
    for (let message of data) {
      if (message.Raw && message.Raw.To) {
        if (message.Raw.To.includes(to)) {
          id = message.ID;
          break;
        }
      }
    }
  }

  if (id) {
    const { data } = await axios.get(`${MAILHOG_HOST}/api/v1/messages/${id}`);
    result = data;
  }
  return result;
};

// tagSelector, see https://cheerio.js.org/
const getLinkFromEmail = (email, tag) => {
  console.log(`getLinkFromEmail ${email} ${tag}`);
  const body = utf8.decode(quotedPrintable.decode(email.Content.Body));
  const html = cheerio.load(body);
  const links = html(tag).find('a');
  const link = links[0];
  const result = link.attribs.href;
  return result;
}

const deleteAllEmails = async () => {
  const { data } = await axios.delete(`${MAILHOG_HOST}/api/v1/messages`);
  return data;
}

module.exports = { getEmail, deleteAllEmails, getLinkFromEmail };
```

Now you can use it like this in your test code 
```
    cy.task('getEmail', email).then(response => {
      expect(response).to.exist;
      cy.log(`Email found ${response.ID}`);

      cy.task('getLinkFromEmail', { email: response, tag: '#button' }).then(url => {
        cy.log('Account confirmation URL', url);
        cy.visit(url);
      })
    });
```

Enjoy!

![hog](/images/2022/hog.png) 
