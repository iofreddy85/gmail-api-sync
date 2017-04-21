# gmail-api-sync

[![npm version](https://badge.fury.io/js/gmail-api-sync.svg)](https://badge.fury.io/js/gmail-api-sync)
[![Build Status](https://travis-ci.org/fedecia/gmail-api-sync.svg?branch=master)](https://travis-ci.org/fedecia/gmail-api-sync)

Sync, query and parse Gmail e-mails with Google API.

Provides full and partial e-mail synchronization, as well as parsing the returned message objects into text and html. Takes care of Google Oauth 2 authentication with convenience methods.

## Installation

```sh
#install and save
npm install gmail-api-sync --save
```

## Usage
Require the module.
```js
var gmailApiSync = require('gmail-api-sync');
```
Load Google Api Project client secret. To get this file follow [Step 1](https://developers.google.com/gmail/api/quickstart/nodejs) .
```js
gmailApiSync.setClientSecretsFile('./client_secret.json');
```
### Full Synchronization (Requires Oauth2 authentication see below)
Query all e-mails (e.g. from:facebook.com, newer_than:2d, before:2017/04/18).
See all posible queries [Search operators you can use with Gmail](https://support.google.com/mail/answer/7190?hl=en)
```js
var options = {query: 'from:facebook.com'};

gmailApiSync.queryMessages(oauth, options, function (err, response) {
  if (!err) {
      console.log(JSON.stringify(response));
 //   {
 //       "emails": [
 //           {
 //               "id": "15a863bf55605f13",
 //               "date": "Tue, 28 Feb 2017 11:39:20 -0800",
 //               "from": "Facebook Analytics for Apps <analyticsforapps-noreply@facebook.com>",
 //               "subject": "New from Facebook Analytics for Apps",
 //               "textHtml": "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional //EN\"><html><head><title>Facebook</title><meta http-equiv=
 //               \"Content-Type\" content=\"text/html; charset=utf-8\" /><style>@media all and (max-width: 480px){*[class].ib_t{min-width:100% !important}*[clas (...)",
 //               "textPlain": "Hi Juan,\r\n\r\nFacebook Analytics for Apps Hits 1 Million Apps, Websites, Bots and Adds New Features\r\n\r\nWe're so (...)",
 //                "historyId": "13855"
 //             }
 //       ],
 //       "historyId": "13855"
 //   }
});
```
### Partial Synchronization (Requires Oauth2 authentication see below)
Sync e-mail changes with historyId from last message retrieved.
For more information on historyId see [Partial synchronization](https://developers.google.com/gmail/api/guides/sync)

```js
var options = {historyId: previousResponse.historyId}; //previousResponse.historyId: "13855"

gmailApiSync.syncMessages(oauth, options, function (err, response) {
  if (!err) {
      console.log(JSON.stringify(response));
 //       {
 //           "emails": [ {
 //               "id": "15b87edac93971d1",
 //               "date": "Wed, 19 Apr 2017 13:35:50 -0700",
 //               "from": "Facebook <notification+kjdvmpu51uu_@facebookmail.com>",
 //               "subject": "Juan, you have 67 new notifications",
 //               "textHtml": "<!DOCTYPE HTML PUBLIC \"-//W3C//DTD HTML 4.01 Transitional //EN\"><html><head><title>Facebook</title><meta http-equiv=\"Content-Type\" content=\"text/html; charset=utf-8\" /><style>@media all and (max-width: 480px){*[class].ib_t{min-width:100% !important}*[class].ib_row{display:block !important}*
 //               "textPlain": "========================================\r\nGo to Facebook\r\nhttps://www.facebook.com/n/?
 //               "historyId": "15424"
 //            }
 //           ],
 //           "historyId": "15424"
 //       }
  }
});
```
### Authentication with Access Token
This assumes you already have an access token from Google Api. To get one, see test [Generate Access Token](../master/test/generate_access_token.js) .
```js
gmailApiSync.authorizeWithToken(accessToken, function (err, oauth) {

});
```

### Authentication with ServerAuth Code
This assumes you already have a ServerAuth Code from Google Api. To get one, see test [Generate Server Auth code](../master/test/generate_access_token.js) .
```js
gmailApiSync.authorizeWithServerAuth(serverAuthCode, function (err, oauth) {

});
```

### Reponse Format
The returned e-mails can be formated in the following modes if specified in the options param:
- **list**: Returns the most basic format including just the messageId and threadId.
- **metadata**: Returns all the message headers but no body (textHtml or textPlain).
- **raw**: Returns the full email message data with body content in the raw field as a base64url encoded string.
- **full** (default): Returns a fully parsed message including all headers and decoded body (textHtml and/or textPlain).
```js
var options = {
                query: 'subject:"Claim your free iPhone"',
                format: 'metadata'
               }
```

### Full working example
```js
var gmailApiSync = require('gmail-api-sync');

gmailApiSync.setClientSecretsFile('./client_secret.json');

var options = {
                query: 'from:*.org',
                format: 'metadata'
              }

gmailApiSync.authorizeWithToken(accessToken, function (err, oauth) {
    if (err) {
        console.log('Something went wrong: ' + err);
        return;
    }
    else {
        gmailApiSync.queryMessages(oauth, options, function (err, response) {
            if (err) {
                console.log('Something went wrong: ' + err);
                return;
            }
            console.log(JSON.stringify(response));
        });
    }
});
```
