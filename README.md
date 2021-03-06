# SendGrid Integration

This is a summary of how to integrate with Sendgrid v3. Or about how I went about it, which may help you do it since there is a lot of documentation on their site
and a few examples out there, but the connections aren't always straight forward.

### Goals:
The goal is to integrate the with API so we get to set up SendGrid and we get it sending e-mails when users register into the application.
This will cover sending emails from templates in SendGrid, as well as specifying tracking settings.

Notes:
The way I am writing the code it's supposed to be set up so you write your scripts and set the calls from a different file.

## Why SendGrid and not Mailchimp?
There are various reasons why, but some of the more important:
1. Sendgrid has integration with Auth0. Auth0 is the tool we were using for authentication therefore we wanted the integration.
2. Sendgrid seems to have a very good intuitive dashboard based on statistics by settings that are personalizable.

## Set up and get an API key

1. Create an account in SendGrid
2. Go to Settings/api_keys and create a key

You are going to need it to connect to it in your application. In this case I am going to set up the API key at the top of the file in a variable.
In an application example, you would save all of your keys in a separate file that gets activated and populates the
keys in `env`. variables to get hold of them in the neccesary scripts.

```javascript
const SENDGRID_API = "********";
```

## Set up Transactional Templates

Transactional templates are different to normal Templates in that in Sendgrid only the first ones will be available to use
through the API in order to send automatically.

1. Set up a transactional template.
2. Copy the id of the template and save it

```javascript
const TRANSACTIONAL_TEMPLATE = "id_number";
```

## Set up all the libraries you need (Sendgrid/client)

In order to be able to do this we need to install `@sendgrid/client` like you can see in package.json file.

In your command line run:

````
npm install --save @sendgrid/client
yarn add @sendgrid/client
````

for more information in this library you can visit this link:
    - https://www.npmjs.com/package/@sendgrid/client

## Set up the client Key

1. Set up the client key with the API key. By doing this globaly within the scope of the file you will be able to avoid a few lines of code.

```javascript
client.setApiKey(SENDGRID_API);
```

## Set up a function that adds your registered users into a list in SendGrid:

You will have to get hold of the url "/v3/contactdb/recipients"

```javascript
export function addUserSendgrid(firstName, lastName, email) {
  const request = {
    method: 'POST',
    url: '/v3/contactdb/recipients'
  };
  const data = [{
    email: email,
    first_name: firstName,
    last_name: lastName
  }];
  request.body = data;
  client.request(request).then(([response, body]) => {
    console.log(response.statusCode);
    console.log(response.body);
  });

  request.write('null');
  request.end();
}
```

## Send a Transactional template to a new user
Once you set up a template and get the id of the template, you would then call the url API '/v3/mail/send';

```javascript
export function send_welcome_email(email) {
  const request = {
    method: 'POST',
    url: '/v3/mail/send'
  };

  // setting up the data for the email
  const data = {
    template_id: TRANSACTIONAL_TEMPLATE,  // Insert your template id here
    from: {
      email: 'email@email.com' // This is the email that will appear on your senders details
    },
    subject: '<Subject of the email>', // Subject of the email
    filters: dataFilters,
    tracking_settings: dataTrackingSettings
  };

  let dataPersonalization = [{
    to: [{
      "<user.email>" // This being the users email
    }],
    category: ['<categories>'],
    filters: dataFilters,
    tracking_settings: dataTrackingSettings
  }];

  data.personalizations = dataPersonalization;
  request.body = data;
  client.request(request).then(
    ([response, body]) => {
      console.log(response.statusCode);
      console.log(response.body);
    },
    error => {
      Raven.captureException(error);
      console.log(error);
    }
  );
}
```

## Set up Settings and filters for tracking

One of the reasons I chose to use Sendgrid as opposed to Mailchimp is because it allows you to configure settings and options.
In this case I am checking whether the user tracks the clicks and the google analytics. There are a few other settings you are able
to set up, for that you should follow this link and discover the documentation and add the options to in the object.

Documentation url: https://sendgrid.api-docs.io/v3.0/contacts-api-custom-fields

```javascript
// Setting tracking email here:
const dataFilters = {
  filters: {
    clicktrack: {
      settings: {
        enable: 1
      }
    },
    dkim: {
      settings: {
        domain: 'youdomain.com',
        use_from: false
      }
    }
  }
};

// Setting tracking and ganalytics
const dataTrackingSettings = {
  click_tracking: {
    enable: true
  },
  open_tracking: {
    enable: true
  },
  ganalytics: {
    enable: true,
    utm_source: 'yourdomain.com'
  }
};
```

## Call your functions

This will be called then from `authentication/yourAuthentication.js`. As you could have figured since we were exporting this functions.

For more information and documentaion, read their docs. This is meant to help put together some simple scripting with their version 3 (v3) of their documentation.

For the complete code go to `sendgrid/sendgrid.js`.
