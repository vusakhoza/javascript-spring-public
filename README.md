# javascript-spring-public
Javascript Spring SDK Issues Repo

The purpose of this repo is to track issues regarding the use of the Spring JS SDK. Find the full docs [here](https://spring.co.zw).

# Documentation

The inspiration behind Spring was to develop a service that makes it easier for developers to access relevant and intiutive payment solutions for both web and mobile platforms. In a market that emphasises technical protocol over user experience we realised that the later always trumps the former, as such we sought to be a breath of fresh air for both devs as well as end users looking for an ecommerce solution that just makes sense.

We currently only have Javascript Web and React Native SDKs. We are working hard to write more SDKs to cater for all the different flavors of development out there.

## Javascript Web SDK
We are a big fan of Javascript because of its versatility. A large chunk of our stack uses Javascript and we recommend devs to adopt it for their web based projects since it's pluggable into just about any web based platform available at the moment. There are many platforms that are built over Javascript in order to make it even more efficient, the likes of jQuery, React, AngularJS and Vue just to mention a few. The Javascript SDK requires none of these to work, its a vanilla plugin that's meant to work on as wide a range of browsers as possible.

### Get an API Key
> All links are still pointing to the dev IP address pending the move to the live server domain name

The first thing you need to do is to get an api key. To do so, [Register](http://207.180.250.156:8500) a developer account on the developer portal. Once you've logged in, [Create](http://207.180.250.156:8500/newapp/) a new project, this will generate a new API Key for your project. You can create as many projects as you like. You will need an API Key to proceed.

### Getting the Access Key
Its worth mentioning at this point that the Spring platform uses a custom [OAuth](https://en.wikipedia.org/wiki/OAuth) mechanism to validate requests for submitting or requesting user data. The process for gaining access to the sdk interfaces can be simplified as follows:

```
> Get api key 
> Get auth token 
> Use auth token + api key to get access key 
> Use access key to invoke interface
```

We've already gone through [getting the api key](#get-an-api-key).

#### Get auth token
The auth token is a 256 bit token used to authenticate requests to certain Spring endpoints. To get the auth token, use the developer account email and username you used to register on the developer portal. We recommend that this be done by a server side function that then passes the token to the client side, this approach is recommended so that the client side source code doesn't contain clear username and passwords. Below is a basic [Python requests](http://docs.python-requests.org/en/master/) example, feel free to translate the execution to a server side language of your choice.

```
import requests

url = "https://spring.co.zw/api/get-token/"
user = "test@test.com"
pass = "testify"

payload = "username=%s&password=%s" % (user, pass)
headers = {
    'Content-Type': "application/x-www-form-urlencoded",
}

response = requests.post(url, data=payload, headers=headers)

print(response.text)
"""
    {
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1NTYxMDEyOTYsImVtYWlsIjoidnVzYWtob3phQGdtYWlsLmNvbSIsInVzZXJuYW1lIjoiYWRtaW4iLCJ1c2VyX2lkIjoxfQ.yyngN8BxiYCAtlIjLkHNM3YClhifF-XL_P5BZAZB3Dw"
    }
"""
});
```
The reponse is a json string with one key value pair. We dont have a strong opinion on how the client obtains the auth token. Feel free to explore secure mechanisms that suite your stack.

### Get Access Key
The endpoint for obtaining the access key requires an `Authorization` header that contains a string in the format 

`Bearer auth_token_goes_here`

The endpoint also expects json data with one key value pair:

```
{
	"aak": "9ddcac12ad79b55e5413cde8c34e367a"
}
```

Below is a basic Python requests example:
```
url = "https://spring.co.zw/api/get-auth/"

payload = {
    "aak": "9ddcac12ad79b55e5413cde8c34e367a"
}
headers = {
    'Content-Type': "application/json",
    'Authorization': "Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1NTYxMDEyOTYsImVtYWlsIjoidnVzYWtob3phQGdtYWlsLmNvbSIsInVzZXJuYW1lIjoiYWRtaW4iLCJ1c2VyX2lkIjoxfQ.yyngN8BxiYCAtlIjLkHNM3YClhifF-XL_P5BZAZB3Dw",
}

response = requests.post(url, json=payload, headers=headers)

print(response.text)
"""
{
    "access_key": "cd9a8c475c974f46f65fff1fd8c1223629b27257a4767ed7ab509f1541981843",
    "timestamp": "2019-04-24T10:35:58.228776",
    "jwt": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1NTYxMDI0NTgsImVtYWlsIjoidnVzYWtob3phQGdtYWlsLmNvbSIsInVzZXJuYW1lIjoiYWRtaW4iLCJ1c2VyX2lkIjoxfQ.r4EU9fFnm26uK5TlaLL1LDjo1nSuNwQVOH0bSuVwihw",
    "application": 3
}
"""
```
The reponse is a json string with a number of key value pairs, the important ones are:
- access_key - the access key to be used for initializing the JS SDK
- jwt - an auth token with a newer timestamp if required

### Load the SDK
In order for you to use the platform you have to include the sdk in your project:

`<script src="https://unpkg.com/@opexaorg/spring@1.0.0/sdk.js" async defer></script>`

<!--
### Add an action button
Add a DOM element that will initiate the chosen Spring action:

`<input id="spring-action" type="button" value="Let's act"/>`
-->

### Initiate SDK
Now that we've obtained an access key and setup our DOM elements. The first step in using the sdk is to run the initialization method. The `init` method takes an object as an argument, the object need only have one value, the access key.

```
var spring = new Spring();
// initialize spring
spring.init({
    access: access_key
})
.then(function(res){
    console.log(res);
    /* 
        {
            data: {
                access_key: "9e5df91a7bc450a21d12504b9707e100ea4c7cf0e9af2c16687e90420bde26d4"
                application: {
                    business_area: "International business"
                    created: "2019-04-22T20:59:21.354498"
                    description: "Mobile POS machine"
                    name: "Metbank Merchant App"
                    state: "Test Checkout"
                }
                token: "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJleHAiOjE1NTYwNTE3NDEsImVtYWlsIjoidnVzYWtob3phQGdtYWlsLmNvbSIsInVzZXJuYW1lIjoiYWRtaW4iLCJ1c2VyX2lkIjoxfQ.vvtzK_H6eSzcwJr1LFGkRtgdoFqQIBq2nZPn_CMBalA"
            }
            success: true // if false then the request was rejected, likely because of invalid api key
        }
    */
})
.catch(function(e){
    console.log('error reason', e);
});
```

The JS sdk uses [Javascript Promises](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/Promise) to manage asynchronous requests. Let look at the important bits of the results from the `init` method:
- data.application - contains application details for the application that generated the access key
- data.token - a token with a more recent timestamp if required
- success - a boolean that indicates whether the access key was valid or not. A `false` result here would mean all subsequent requests will be rejected.

### API actions
Once initialized the sdk can now be used to call the `api` method. The `api` method also takes an object as an argument. The object can have the following fields depending on the action indicated. This is what the object expected by the `api` method can look like:
```
{
    action: 'pay' | 'merchant' | 'login' | 'register' | 'cards', // required for all requests
    client_reference: 'SPR123', // @string, user generated alpha numeric string, required for pay, merchant actions
    total_amount: 12345, // @number, amount is in cents, required for pay, merchant actions
    sdk: 'mobile', // @string, required for all requests
    merchant_phone: '263776853857' // @string, a merchant's Spring registered phone number
}
```
To find out more about the concept behind actions, refer to the conceptualization page.

### Making an action request
We refer to the process of invoking a client interface as an 'action'. As such, an action request is a request to invoke a client interface. Upon approval the sdk launches a new window and loads the requested client interface in that window. A listener is also launched that listens for the results of the transaction. Developers can then listen for results and behave accordingly upon transaction completion. Below this is an example api call for a 'pay' action.

```
spring.api({
    action: 'pay',
    client_reference: 'SPR123',
    total_amount: 12345, // amount is in cents
    sdk: 'mobile',
    merchant_phone: '263776853857'
})
.then(function(res){
    console.log(res);
    /*
    {
        message: "Authorised"
        // payload from Spring will defer depending on the action invoked
        payload: { 
            action: "pay"
            client_phone: "263773591219"
            client_reference: "SPR123"
            gateway_reference: 693711
            institution: "FBC"
            pan: "533303xxxxxx4141"
            timestamp: "Apr 24, 2019 13:44 PM"
            total_amount: "12345"
        }
        success: true // if false then the request was rejected
    }
    */
})
.catch(function(e){
    console.log('error reason', e);
});
```
The reponse contains transaction details for the developers records as well as to use to verify if the transaction matches the original sent to Spring. It is also important to check the `res.payload.action` variable to see if it matches the action that was specified in the invocation since users at times can be prompted to go through other actions like login or registration before the actual intended action. Such actions will also send data to the callback function. Responses from action will however always have an action field to assist developers to distinguish them.
