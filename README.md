BitBalloon Node Client
======================

BitBalloon is a hosting service for the programmable web. It understands your documents, processes forms and lets you do deploys, manage forms submissions, inject javascript snippets into sites and do intelligent updates of HTML documents through it's API.

Installation
============

Install by running

    npm install bitballoon


Authenticating
==============

You'll need an application client id and a client secret before you can access the BitBalloon API. Please contact us at team@bitballoon.com for your credentials.

Once you have your credentials you can instantiate a BitBalloon client.

```js
bitballoon = require("bitballoon");

client = bitballoon.createClient({access_token: OAUTH_ACCESS_TOKEN});
```

You'll need an access_token before you can use the client. Often you'll have this stored in your database, but the bitballoon client lets you acquire an access token in two ways.

1. **Authorize from credentials**: Authenticate directly with your API credentials.
2. **Authorize from a URL**: send a user to a URL, where she can grant your access API access on her behalf.

The first method is the simplest, but only works when accessing with your own BitBalloon users' permissions is what you need:

```js
bitballoon.tokenFromCrendentials({client_id: CLIENT_ID, client_secret: CLIENT_SECRET}, function(err, access_token) {
  return console.log(err) if (err)
  console.log(access_token);
});
```

To authorize on behalf of a user, you first need to send the user to a BitBalloon url where she'll be asked to grant your application permission. Once the user has visited that URL, she'll be redirected back to a redirect URI you specify (this must match the redirect URI on file for your Application). When the user returns to your app, you'll be able to access a `code` query parameter, that you can use to obtain the final `access_token`

```js
url = client.authorizeUrl({redirectUri: "http://www.example.com/callback"});


client.authorize({code: CODE_FROM_QUERY_PARAM, redirectUri: "http://www.example.com/callback"}, function(err, access_token) {
  if (err) return console.log(err);
  console.log(access_token);
});

```

Command Line Utility
====================

The BitBalloon gem comes with a handy command line utility for deploying and redeploying sites.

To deploy the site in the current working directory:

```
bitballoon deploy
```

The first time you deploy, you will be asked for your `client id` and `client secret`. After the deploy the tool will store an `access_token` and the `site_id` in `.bitballoon`. Next time you run the command the tool will redeploy the site using the stored `access_token`.

You can also deploy a specific path:

```
bitballoon deploy /path/to/my/site
```

Or a zip file:

```
bitballoon deploy /path/to/my/site.zip
```

Sites
=====

Getting a list of all sites you have access to:

```js
client.sites(function(err, sites) {
  // do work
});
```

Getting a specific site by id:

```js
client.site(id, function(err, site) {
  // do work
})
```

Creating a site from a directory:

```js
client.createSite({dir: "/tmp/my-site"}, function(err, site) {
  // do work
});
```

Creating a site from a zip file:

```js
client.createSite({zip: "/tmp/my-site.zip"}, function(err, site) {
  // do work
});
```

Both methods will create the site and upload the files. The site will then be processing.

```js
client.createSite({zip: "/tmp-my-site.zip"}, function(err, site) {
  site.state == "processing"
});
```

Refresh a site to update the state:

```js
site.refresh(function(err, site) {
  console.log(site.state);
});
```

Use `waitForReady` to wait until a site has finished processing.

```js
client.createSite({zip: "/tmp-my-site.zip"}, function(err, site) {
  site.waitForReady(function(err, site) {
    if (err) return console.log("Error deploying site %o", err);
    console.log("Site deployed");
  });
});
```

Redeploy a site from a dir:

```js
client.site(id, function(err, site) {
  if (err) return console.log("Error finding site %o", err);
  site.update({dir: "/tmp/my-site"}, function(err, site) {
    if (err) return console.log("Error updating site %o", err);
    site.waitForReady(function(err, site) {
      if (err) return console.log("Error updating site %o", err);
      console.log("Site redeployed");
    });
  });
})
```

Redeploy a site from a zip file:

```ruby
client.site(id, function(err, site) {
  if (err) return console.log("Error finding site %o", err);
  site.update({zip: "/tmp/my-site.zip"}, function(err, site) {
    if (err) return console.log("Error updating site %o", err);
    site.waitForReady(function(err, site) {
      if (err) return console.log("Error updating site %o", err);
      console.log("Site redeployed");
    });
  });
})
```

Update the name of the site (its subdomain), the custom domain and the notification email for form submissions:

```js
site.update({name: "my-site", customDomain: "www.example.com", notificationEmail: "me@example.com"}, function(err, site) {
  if (err) return console.log("Error updating site %o", err);
  console.log("Updated site");
});
```

Deleting a site:

```js
site.destroy(function(err) {
  if (err) return console.log("Error deleting site");
  console.log("Site deleted");
});
```

Forms
=====

Access all forms you have access to:

```js
client.forms(function(err, forms) {
  // do work
})
```

Access forms for a specific site:

```js
client.site(id, function(err, site) {
  if (err) return console.log("Error getting site %o", err);
  site.forms(function(err, forms) {
    // do work
  });
});
```

Access a specific form:

```js
client.form(id, function(err, form) {
  if (err) return console.log("Error getting form %o", err);
  // do work
});
```

Access a list of all form submissions you have access to:

```js
client.submissions(function(err, submissions) {
  if (err) return console.log("Error getting submissions %o", err);
  // do work
});
```

Access submissions from a specific site

```js
client.site(id, function(err, site) {
  if (err) return console.log("Error getting site %o", err);
  site.submissions(function(err, submissions) {
    if (err) return console.log("Error getting submissions %o", err);
    // do work
  })
});
```

Access submissions from a specific form

```js
client.form(id, function(err, form) {
  if (err) return console.log("Error getting form %o", err);
  form.submissions(function(err, submissions) {
    if (err) return console.log("Error getting submissions %o", err);
    // do work
  });
});
```

Get a specific submission

```js
client.submission(id, function(err, submission) {
  if (err) return console.log("Error getting submission %o", err);
  // do work
})
```

Files
=====

Access all files in a site:

```js
client.site(id, function(err, site) {
  if (err) return console.log("Error getting site %o", err);
  site.files(function(err, files) {
    if (err) return console.log("Error getting files %o", err);
    // do work
  });
});
```

Get a specific file:

```js
client.site(id, function(err, site) {
  if (err) return console.log("Error getting site %o", err);
  site.file(path, function(err, file) {
    if (err) return console.log("Error getting file %o", err);
    
    file.readFile(function(err, data) {
      if (err) return console.log("Error reading file %o", err);
      console.log("Got data %o", data);
    });
    
    file.writeFile("Hello, World!", function(err, file) {
      if (err) return console.log("Error writing to file %o", err);
      console.log("Wrote to file - site will now be processing");
    });
  });
});
```

Snippets
========

Snippets are small code snippets injected into all HTML pages of a site right before the closing head or body tag. To get all snippets for a site:

```js
client.site(id, function(err, site) {
  if (err) return console.log("Error getting site %o", err);
  site.snippets(function(err, snippets) {
    if (err) return console.log("Error getting snippets %o", err);
    // do work
  });
});
```

Get a specific snippet

```js
client.site(id, function(err, site) {
  if (err) return console.log("Error getting site %o", err);
  site.snippet(snippetId, function(err, snippet) {
    if (err) return console.log("Error getting snippet %o", err);
    // do work
  });
});
```