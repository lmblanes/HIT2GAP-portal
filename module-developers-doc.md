# HIT2GAP WebPortal Prototype Documentation – To the attention of module developers.


## Loading of the module
The HIT2GAP WebPortal Prototype is reachable at:

```
https://portal.h2g-platform-core.nobatek.com/
```

After loading, the portal checks the user authentication (ie. Authenticated to the HIT2GAP Platform SSO) :

1. If the user is logged in, go to 3

2. If the user is **not** logged in, the log-in form will appear and once the user is authenticated, go to 3.

3. **Once authenticated** modules are available through a dropdownlist in the navigation bar, showing module name pointing to its ```MODULE_URL```.
Where ```MODULE_URL``` is the full url of your module (including the “https://” part).

When the user chooses a module name in the dropdown list, the platform will load the choosen module in the iframe by requesting the ```MODULE_URL```, on which the ```h2gtokenid``` parameter has been appended.


## Handling the ```h2gtokenid``` parameter
As the ```MODULE_URL``` is called with the appended ```h2gtokenid``` parameter, you’ll need to get that ```h2gtokenid``` parameter and make your HIT2GAP Platform API calls with that token in a header parameter called ```Cookie```, with the value ```session=``` concatenated with the value of the ```h2tokenid``` parameter to perform fully authenticated requests to the API.

**Important:** in order the portal to be fully functional, **your module** has to be **reachable through secured http (https://) protocol, otherwise your module won’t load in the iframe**.

### Examples

#### Public-side (common to all the server-side interpretations) :
Let’s assume that a module is reachable at this address: ```https://my_hit2gap_module.com```

The portal navigation bar will contain an entry with this url.
When clicked and if or once the user is authenticated, the portal will load the module in its iframe with the following URL: ```https://my_hit2gap_module.com?h2gtokenid=123456ABCDEF```.


#### Server-side code samples
You need to make your calls to the platform API by adding an header value :

##### cURL Command Only:
```
curl --header "Cookie: session=$h2gtokenid" https://.../api/timeseries/ABCDEF123456?start_time=2018-02-23T00:00:00Z&end_time=2018-02-24T00:00:00Z
```


##### PHP:
Server-side example in PHP with GuzzleHttp Library:

```php
// Getting the authentication token
$h2gtoken = $_GET['h2gtokenid'];

// Make a call to the API with session token as header parameter (using guzzlehttp library)
$client->request('GET', 'https://.../api/timeseries/ABCDEF123456?start_time=2018-02-23T00:00:00Z&end_time=2018-02-24T00:00:00Z', [
    'headers' => [
        'Cookie' => 'session='.$h2gtoken
    ]
]);

// Process response...
```


##### Python (with “Flask“ and “Requests” libraries):

```python
from flask import request
import requests

# get authentication token
h2gtoken = request.args.get('h2gtokenid')
# build API request cookies
cookies = {'session': h2gtoken}
# send request to timeseries API
url = 'https://.../api/timeseries/ABCDEF123456'
params = {
    'start_time': '2018-02-23T00:00:00Z',
    'end_time': '2018-02-24T00:00:00Z',
}
try:
    response = requests.post(url, cookies=cookies, params=params)
except (requests.ConnectionError, requests.Timeout,) as exc:
    # process exception...
# process response...
```


## Getting authenicated user information
The ```h2gtokenid``` does not give you any informative data about the connected user.

That's why the JavaScript postMessage API (https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage) has been implemented on the portal and listens for the ```whoami``` message.
Once the message is received by the platform, it sends back a postMessage to the HTML page loaded in the iframe, with the requested data.

That means that, on the module side, you need to :
1. Send the ```whoami``` postMessage to request the information
2. Listen for a message from the portal (parent frame) to handle the data.

### JavaScript Sample Code
#### Sending the ```whoami``` message to the portal
```javascript
parent.postMessage('whoami', document.location.origin);
```

#### Listening for the ```whoami``` response from the portal
```javascript
window.addEventListener('message', function(event) {
    var info = JSON.parse(event.data);
    switch (info.responseToMessage) {
        case 'whoami':

            //...
            //HERE IS YOUR CODE MANAGING THE RECEIVED DATA
            //...

            break;
        default:
            console.warn('No response, message unknown: ' + info.responseToMessage);
            break;
    }
});
```


### JavaScript whoAmi() implementation
In order to facilitate the integration of that postMessage() process, you'll find at the following URL a fully functional sample code allowing you to only have to call a ```whoAmI()``` function with a callback function as argument.

https://portal.h2g-platform-core.nobatek.com/static/js/module.js

```javascript
var whoAmICallback = null;


$(document).ready(function() {

    window.addEventListener('message', function(event) {
        var info = JSON.parse(event.data);
        switch (info.responseToMessage) {
            case 'whoami':
                if (whoAmICallback != null) {
                    whoAmICallback(info.identity);
                }
                break;
            default:
                console.warn('No response, message unknown: ' + info.responseToMessage);
                break;
        }
    });

});


function whoAmI(callback) {
    if (callback != null) {
        whoAmICallback = callback;
        parent.postMessage('whoami', document.location.origin);
    }
    else {
        console.warn('whoAmI callback is undefined!');
    }
}
```

#### Example of whoAmI() use
##### With anonymous function as callback argument
```javascript
whoAmI(function(identify){
   console.log(identify);
});
```

##### With identified function as callback argument
```javascript
function handlingIdentity(identity){
    console.log(identity);
}

whoAmI(handlingIdentity);
```
