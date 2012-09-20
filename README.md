# 500px JavaScript SDK

The JavaScript SDK allows you to use the [500px API](https://github.com/500px/api-documentation) through browser-side JavaScript.

[Click here to view a demo of the JavaScript SDK](http://500px.github.com/500px-js-sdk/)<br>
[View the source for the demo](https://github.com/500px/500px-js-sdk/blob/master/examples/example1.html)
## Setup

### Configuring Callback

In order to process cross-domain API requests through JavaScript you will need to upload a `callback.html` file to your site. The `callback.html` file must be hosted on the same domain and protocol as the page that will be using the API. For example if your application will be located at `http://example.com/` then your callback can be located at `http://example.com/callback.html`. The filename and sub directory are not important so for example this would also work: `http://example.com/500px_sdk/callback_file.html`. If your callback is on a different protocol, domain or port number then you will not be able to process the cross-domain requests and the SDK will not work.

Once you have placed the callback file you will need to enter the URL of the callback file in your Application settings. To do this go to http://500px.com/settings/applications, create a new application or edit an existing one, and in the JavaScript SDK Callback field enter the URL to your `callback.html` file.

### 500px.js

The next step is to include the `500px.js` file in your application. For example:

    <script src="http://example.com/500px.js"></script>

### Initialize the SDK

To initialize the SDK you will need your SDK key. This can be found on http://500px.com/settings/applications.
Click the *See application details* link for your application and copy the value given for *JavaScript SDK Key*. In your application use the `init` method to initialize the SDK:
```html
<script>
  _500px.init({
    sdk_key: 'YOUR_SDK_KEY'
  });
</script>
```
    
If you're using jQuery (or a similar framework) you might want to wait until the page is loaded:

```html
<script>
  $(document).ready(function () {
    _500px.init({
      sdk_key: 'YOUR_SDK_KEY'
    });
  });
</script>
```

## Using the SDK

### Making API Calls

To make an API call use the `api` function.

```javascript
_500px.api('/photos', { feature: 'popular', page: 1 }, function (response) {
    console.log(response.data.photos);
});
```

To see a list of available API calls, the parameters they take and what they return see the [API Documentation](https://github.com/500px/api-documentation).

### The Response Object

Calls to `api` return a `Response` object. The response object has these properties:

- `success` (boolean) True if no errors occurred.
- `error` (boolean) True if an error occurred.
- `error_message` (string) If an error occurred, this value will have the error message.
- `status` (integer) The HTTP status code of the response.
- `data` (object) The data returned by the API call.

Examples:

```javascript
_500px.api('/photos', {feature: 'popular', page: 1}, function (response) {
    if (response.success) {
        alert('There are ' + response.data.photos.length + ' photos.');
    } else {
        alert('Unable to complete request: ' + response.status + ' - ' + response.error_message);
    }
});
```

This example sends a PUT request to update a photo:
```javascript
_500px.api('/photos/8888888', 'put', {name: 'New Photo Name'}, function (response) {
   if (response.success) {
    alert('You photo was updated.')
   } 
});
```

### Logging a user in

Certain API calls do not require a user to approve your application (for example getting a list of popular photos). In the [API Documentation](https://github.com/500px/api-documentation) these calls are labelled as requiring only a consumer key. For calls that require an Access Token you will need to authenticate the user first.
To authenticate a user use the `login` method. If you pass it a callback function it will be called with the status of the login. The status can have two values:
- `authorized` The user authorized your app. You now have an Access Token for this user.
- `denied` The user denied your application. You do not have an Access Token for this user.

```javascript
_500px.login(function (status) {
    if (status == 'authorized') {
        alert('You have logged in');
        _500px.api('/users', function (response) {
            // .......
        });
    } else {
        alert('You denied my application');
    }
});
```

### getAuthorizationStatus

Each time you call `login` a popup window is opened where the user can login and authorize your application. If the user has already logged in and authorized your app then calling `login` will open the popup, and the popup will close again immediately. This is because 500px will not require a user to re-authorize an application. To avoid the popup opening unnecessarily you can use the `getAuthorizationStatus` method to check if the user needs to login.

`getAuthorizationStatus` can be passed a callback that will be given a string with the status of the user's sessions. The value of the status can be:
- `not_logged_in` The user is not logged in. You should call `login` to log the user in.
- `not_authorized` The user is logged in but has not authorized your application. You should call `login` so the user can authorize it.
- `authorized` The user has authorized your application. An access token has been obtained for the user.

Example:
```javascript
_500px.getAuthorizationStatus(function (status) {
    if (status == 'not_logged_in' || status == 'not_authorized') {
        _500px.login();
    }
});
```

### ensureAuthorization

`ensureAuthorization` will check if a user has authorized your application, and prompt them to login/authorize it if they haven't. It can be passed a callback function that will be called if/when the user has authorized your application. If an access token has already been obtained for the user using `login` or `getAuthorizationStatus` the callback will be executed immediately.

Example:
```javascript
_500px.ensureAuthorization(function () {
    _500px.api('/users', function (response) {
        alert("Your username is " + response.user.username);
    });
});
```

## Events

There are three events that can be subscribed to:
- `authorization_obtained` Triggered when an Access Token is obtained for the user. `login`, `getAuthorizationStatus` and `ensureAuthorization` will trigger this event when they obtain an access token for a user.
- `authorization_denied` Triggered when the user is prompted to authorize your application and declines to grant authorization.
- `logout` Triggered when the user logs out, or if there is an OAuth error (for example the Access Token you have is no longer valid).

### Subscribing to events

Use the `on` function to subscribe to an event.

```javascript
_500px.on('authorization_obtained', function () {
    alert('You have logged in');
});
```