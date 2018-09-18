# HIT2GAP WebPortal Prototype Documentation – To the attention of module developers.

The HIT2GAP WebPortal Prototype is reachable at:

```
https://portal.h2g-platform-core.nobatek.com/
```

Once authenticated modules are available through a dropdownlist in the navigation bar, showing module name pointing to its ```MODULE_URL```.
Where ```MODULE_URL``` is the full url of your module (including the “https://” part).

**Important:** if your module's url contains a query string (...?param1=value1&param2=value2), please replace the "&" sign by its ASCII encoding value "%26".
For example: ```https://h2g-module.com?param=value&item=4``` will become ```https://h2g-module.com?param=value%26item=4```.

By requesting your module through this URL, the portal will:

1.	Check if the user is already logged in (ie. Authenticated to the HIT2GAP Platform SSO)

2. If not: the log-in form will appear and once the user is authenticated, go to 3.

3. If yes: the HIT2GAP WebPortal Prototype will load your module in the iframe and will append the SSO Authentification Token to the ```MODULE_URL``` you provided in a query parameter called “h2gtokenid”.

On the module side (your side), you’ll need to get that “h2gtokenid” parameter and call the HIT2GAP Platform API with that token in a header parameter called “Cookie”, with the value “session=” concatenated with the value of the “h2gtokenid” parameter to perform fully authenticated requests to the API.

**Important:** in order the portal to be fully functional, your module has to be reachable through secured http (https://) protocol, otherwise your module won’t load in the iframe.

## Example
Here is an example to illustrate with realistic values:

### Public-side (common to all the server-side interpretations) :
Let’s assume that a module is reachable at this address: https://my_hit2gap_module.com

The portal navigation bar will contain an entry with this url.
When clicked and if or once the user is authenticated, the portal will load the module in its iframe with the following URL: ```https://my_hit2gap_module.com?h2gtokenid=123456ABCDEF```.


### Server-side code samples
#### PHP:
Server-side example in PHP with GuzzleHttp Library:

Getting the variable:
```php
$h2gtoken = $_GET['h2gtokenid'];
```

Make a call to the API with session token as header parameter (using guzzlehttp library):
```php
$client->request('GET', '/timeseries/NUIG_Network1_No_09_Metering_SDB_CEO6_Elec_Meter_Accum_Total_DATALOG16?start_time=2018-02-23T00:00:00Z&end_time=2018-02-24T00:00:00Z', [
    'headers' => [
        'Cookie' => 'session='.$h2gtoken
    ]
]);
```



#### cURL Command Only:
```
curl --header "Cookie: session=$h2gtokenid" https://.../timeseries/NUIG_Network1_No_09_Metering_SDB_CEO6_Elec_Meter_Accum_Total_DATALOG16?start_time=2018-02-23T00:00:00Z&end_time=2018-02-24T00:00:00Z
```



#### Python (with “Flask“ and “Requests” libraries):

```
from flask import request
import requests

# get authentication token
h2gtoken = request.args.get('h2gtokenid')
# build API request headers
headers = {'Cookie': 'session={}'.format(h2gtoken)}
# send API request
url = 'https://.../timeseries/NUIG_Network1_No_09_Metering_SDB_CEO6_Elec_Meter_Accum_Total_DATALOG16?start_time=2018-02-23T00:00:00Z&end_time=2018-02-24T00:00:00Z'
response = requests.post(url, headers=headers)
# process response...
```
