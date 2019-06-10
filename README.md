# Postman Rsa Signature Support

I have managed to figure out how to include RSA Signature Support within Postman.

The code depends on a third party javascript library: https://kjur.github.io/jsrsasign/jsrsasign-all-min.js

It all feels a bit hacky, but this was the only way i could meet my requirements as Postman does not currently have support for RSA.

## My problem?

Create a RSA signature from the JSON request body using a RSA SHA256 Private Key, append the signature as a custom HTTP header and finally perform a POST operation to a WebApi 2 controller action.

## Solution?
* Using Postman pre-request script, download a third party javascript library and store it as a Postman global variable.
* **IMPORTANT:** after retrieving the javascript, i had to run the following command `eval(postman.getGlobalVariable("rsaLibrary"));` to give my browser access to the library.
* Then i read the request body through `request.data` and serialised it using `JSON.stringify()`
* I think retrieved the RSA private key, that i stored as a Postman Environment Variable using `postman.getEnvironmentVariable("Pem");`
* After this, i passed all of my information to the third party javascript library and executed the `sign()` function.
* I stored the output in a variable and set the header using `pm.request.headers.add()`

My code example is stored below.

# Full Code Example

```javascript
var Header = require('postman-collection').Header;

//Do not remove, the rsaLibrary depends on these objects and they must be instantiated before using the rsaLibrary object.
var navigator = {};
var window = {};

if(!pm.globals.has("rsaLibrary")){
    
    pm.sendRequest("https://kjur.github.io/jsrsasign/jsrsasign-all-min.js", function (err, res) {

    if (err){
        console.log(err);
    }
    else {
        pm.globals.set("rsaLibrary", res.text());
    }})
}

eval(postman.getGlobalVariable("rsaLibrary"));

var signingObject = JSON.stringify(request.data);

console.log("Data to Sign: " + signingObject);

const private_key = postman.getEnvironmentVariable("Pem");

console.log("Private Key: " + private_key);

var signatureLib = new KJUR.crypto.Signature({"alg": "SHA256withRSA"});

signatureLib.init(private_key);
signatureLib.updateString(signingObject);

var signatureHash = signatureLib.sign();

console.log("Signature: " + signatureHash);

//Add payload signature header.
pm.request.headers.add(new Header("X-HUB-Signature: " + signatureHash));

pm.globals.unset("rsaLibrary");
```

Hope this helps someone in the future.
