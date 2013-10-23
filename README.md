# node-hpka

------------------------------------------------------------

A Node.js implementation of the [HPKA protocol](https://github.com/Tashweesh/hpka). Acts as an [expressjs](https://github.com/visionmedia/express) middleware (as of now). A middlware for the standard http stack will come later.

## Installation

To install node-hpka, simply run from app's root folder :
```
npm install hpka
```

## Dependencies

This module depends on [node-cryptopp](https://github.com/Tashweesh/node-crytopp), which itself depends on the [Crypto++](http://cryptopp.com) cryptography library. 

## Usage

As said above, this module as of now acts as a express middleware. The module exposes 2 methods :

**```hpka.middleware(loginCheck, registration)```**: the middleware builder. The two parameters are functions, that aew called once the signature of the request is verified (with the public key attached to it). These functions are called to do the additional handling (like checking the DB for the right public key or register the new user) before handing the request to the correct route. These functions will receive the following parameters:

* loginCheck(HPKAReq, res, callback(Boolean)) :
	* HPKAReq : the HPKAReq object, as descibed below
	* res : the [expressjs res](http://expressjs.com/api.html#res.status) object, in case you want to show some error pages and what not
	* callback(isValid) : function to be called if you didn't respond to the client. isValid should be a boolean indicating whether the user is registered and the public key is valid or not
* registration(HPKAReq, res) :
	* HPKAReq : the HPKAReq object, as described below
	* res : the [expressjs res](http://expressjs.com/api.html#res.status) object, to show a welcome page (or back to the site's homepage).
	
Note that :

* you must use either ```res``` or ```callback``` in loginCheck
* but you **must** use ```res``` in registration

**```hpka.verifySignature(reqBlob, signature, callback(isValid, username, HPKAReq))```**: checks the signature of a HPKA request. To be used if you choose to manage signature verification by yourself

* reqBlob : the content of the "HPKA-Req" header
* signature : the content of the "HPKA-Signature" header
* callback(isValid, username, HPKAReq) : a function called after the signature verification
	* isValid : a boolean, true if the signature is valid, false if it isn't (thanks Captain Obvious)
	* username : username found in the reuquest
	* HPKAReq : the HPKAReq object extracted from the HPKA-Req header  
	
#### HPKAReq object

The HPKAReq object is the result of parsing the [HPKA-Req field](https://github.com/Tashweesh/hpka#hpka-req-protocol). I will here just list its attributes :

* username: a string
* actionType : a number, as defined in the spec
* keyType : a string (either "ecdsa", "rsa" or "dsa")
* In case of keyType == "ecdsa"
	* point : the public ECDSA point, which has :
		* x : the x coordinate, as a hex string
		* y : the y coordinate, as a hex string
* In case of keyType == "rsa"
	* modulus : a hex string of the RSA modulus
	* publicExponent : a hex string of the RSA public exponent
* In case of keyType == "dsa"
	* primeField : a hex string of the DSA prime field
	* divider : a hex string of the DSA divider
	* base : a hex string of the DSA base
	* publicElement : a hex string of the public DSA element 

#### Example

	var hpkaBuilder = require('hpka');
	var hpkaMiddleware = hpkaBuilder.middlware(
		function(HPKAReq, res, callback){
			//If there isn't a user with the username as in HPKAReq, then render error or warning page and return
			//If there is a user with username and publickey as in HPKAReq, then callback(true)
			//Else callback(false)
		},
		function(HPKAReq, res){
			//Save the details from HPKAReq
			//Use res to show welcome page
		},
	);

#### Where to load the expressjs middleware?

To load express middlwares, you have to call the ```app.use(middlware)``` method. The thing is that, as far as I understand, the loading order of middlwares impact the order of middlware execution when a request comes to the server. It seems it is "first loaded, first executed". Also it seems that my middlware isn't called when it is loaded after ```app.router```. In addition, if you want to serve public static folders (without authentication), you should take that into account too. So here is how I loaded the hpka middleware in my example application :

```
app.use(someMiddlware)
app.use(someOtherMiddlware)
…
app.use(somePublicFolder)
app.use(hpkaMiddlware)
app.use(app.router)
```

## License
This module is released under GPLv2.