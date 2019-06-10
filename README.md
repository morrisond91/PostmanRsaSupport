# PostmanRsaSupport

I have managed to figure out how to include RSA Signature Support within Postman.

The code depends on a third party javascript library: https://kjur.github.io/jsrsasign/jsrsasign-all-min.js

```javascript
var Header = require('postman-collection').Header;

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
