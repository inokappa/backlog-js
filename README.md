# backlog-js
Backlog API version 2 client for browser and node.

## Required reading
Please check out the [Nulab Developers portal page](http://developer.nulab-inc.com/docs/backlog/api/2/).


## Installation
NPM:
``` sh
$ npm install --save backlog-js
```

Typings:
```
$ typings i dt~backlog-js -SG
```

## Getting started
Append your "API Key" or "OAuth2 Access Token" to requests to the API to return data.

``` javascript
import 'isomorphic-form-data';
import 'isomorphic-fetch';
import * as es6promise from 'es6-promise';
import * as backlogjs from 'backlog-js';

es6promise.polyfill();

// 'xxx.backlog.jp' or 'xxx.backlogtool.com' or 'your package host'
const host = 'yourSpaceHost';
const apiKey = 'yourApiKey';
const accessToken = 'yourAccessToken';

// Use API Key
const backlog = new backlogjs.Backlog({ host, apikey });

// Use OAuth2 Access Token
const backlog = new backlogjs.Backlog({ host, accessToken });

// Returns information about your space.
backlog.getSpace().then(data => {
  console.log('space:', data);
}).catch(err => {
  console.log('error:', err.message);
});

```

### Example

#### Download File

``` javascript
backlog.getSpaceIcon().then(data => {
    
  // node
  data.body.pipe(fs.createWriteStream(`./${data.filename}`));
    
  // browser
  data.blob().then((blob) => {
    const objectURL = URL.createObjectURL(blob);
    const element = window.document.querySelector('#image');
    element.src = objectURL;
  });
          
}).catch(err => {
  console.log('error:', err.message);
});
```

#### Upload File

``` javascript
// node
const formData = new FormData();
formData.append("filename", "sample.png");
formData.append("file", fs.createReadStream("./sample.png"));

// browser
const $form = window.document.querySelector('#upload_form');
const formData = new FormData($form);

backlog.postSpaceAttachment(formData).then(data => {
  console.log('success:', data);
}).catch(err => {
  console.log('error:', err.message);
});
```

## Use OAuth2 for authentication

Details on the OAuth2 process are available [here](https://developer.nulab-inc.com/docs/backlog/auth#oauth2).

Here are the basic steps for OAuth2 using the Express:
```` javascript
import * as express from 'express';
import * as backlogjs from 'backlog-js';

const app = express();

const host = 'xxx.backlog.jp'; // or 'xxx.backlogtool.com' or 'your package host'
const clientId = 'yourClientId';
const clientSecret = 'yourClientSecret';
const redirectUri = 'https://localhost:3000/callback';
const state = 'yourState';

const credentials = { clientId, clientSecret }
const oauth2 = new backlogjs.OAuth2(credentials);

const authorizationURL = oauth2.getAuthorizationURL({ host, redirectUri, state });

app.get('/auth', (req, res) => {
  res.redirect(authorizationURL);
});

app.get('/callback', async (req, res) => {
  const code = req.query.code;
  try {
    const accessToken = await oauth2.getAccessToken({ host, code, redirectUri });
    console.log('Access Token:', accessToken);
    
    // save access token.
    
    const myself = await new backlogjs.Backlog({
      host, accessToken: accessToken.access_token
    }).getMyself();
    console.log('Myself:', myself);
    
    res.redirect('/');
  } catch (e) {
    console.log('Access Token Error', e.message);
    res.redirect('/login');
  }
});

app.get('/', (req, res) => {
  res.send('Hello');
});

app.get('/login', (req, res) => {
  res.send('<br><a href="/auth">Log in with Backlog</a>');
});

app.listen(3000);

console.log('Express server started on port 3000');
````

## License

MIT License

* http://www.opensource.org/licenses/mit-license.php
