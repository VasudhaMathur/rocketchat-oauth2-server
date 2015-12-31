# oauth2-server

This package is a implementation of the package [node-oauth2-server](https://github.com/thomseddon/node-oauth2-server) for Meteor.
Implement the `athorization_code` and work like the Facebook's OAuth popup.

## Install
```
meteor add rocketchat:oauth2-server
```

## Implementation

### Server implementation
 * Initialize the lib
 * Add routes to the default router
 * Implement an authenticated route

`server/oauth2server.js`
```javascript
var oauth2server = new OAuth2Server({
  // You can change the collection names, the values
  // below are the default values.
  accessTokensCollectionName: 'oauth-access-tokens',
  refreshTokensCollectionName: 'oauth-refresh-tokens',
  clientsCollectionName: 'oauth-clients',
  authCodesCollectionName: 'oauth-auth-codes',
  // You can enable some logs too
  debug: true
});

// Add the express routes of OAuth before the Meteor routes
WebApp.rawConnectHandlers.use(oauth2server.app);

// Add a route to return account information
oauth2server.routes.get('/account', oauth2server.oauth.authorise(), function(req, res, next) {
  var user = Meteor.users.findOne(req.user.id);

  res.send({
    id: user._id,
    name: user.name
  });
});
```

### Client/Pupup implementation

`client/authorize.js`
```javascript
// Define the route to render the popup view
FlowRouter.route('/oauth/authorize', {
  action: function(params, queryParams) {
    BlazeLayout.render('authorize', queryParams);
  }
});

// Subscribe the list of already authorized clients
// to auto accept
Template.authorise.onCreated(function() {
  this.subscribe('authorizedOAuth');
});

// Get the login token to pass to oauth
// This is the best way to identify the logged user
Template.authorise.helpers({
  getToken: function() {
    return localStorage.getItem('Meteor.loginToken');
  }
});

// Auto click the submit/accept button if user already
// accepted this client
Template.authorise.onRendered(function() {
  var data = this.data;
  this.autorun(function(c) {
    var user = Meteor.user();
    if (user && user.oauth && user.oauth.athorizedClients && user.oauth.athorizedClients.indexOf(data.client_id()) > -1) {
      c.stop();
      $('button').click();
    }
  });
});
```

`client/authorize.html`
```html
<template name="authorize">
  {{#if currentUser}}
    <form method="post" action="" role="form" class="{{#unless Template.subscriptionsReady}}hidden{{/unless}}">
      <h2>Authorise</h2>
      <input type="hidden" name="allow" value="yes">
      <input type="hidden" name="token" value="{{getToken}}">
      <input type="hidden" name="client_id" value="{{client_id}}">
      <input type="hidden" name="redirect_uri" value="{{redirect_uri}}">
      <input type="hidden" name="response_type" value="code">
      <button type="submit">Authorise</button>
    </form>
    {{#unless Template.subscriptionsReady}}
      loading...
    {{/unless}}
  {{else}}
    {{> loginButtons}}
  {{/if}}
</template>
```

`client/style.css`
```css
.hidden {
  display: none;
}
```
