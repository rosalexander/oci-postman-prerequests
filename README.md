# Helpful Links
[OCI API Reference](https://docs.cloud.oracle.com/iaas/api/#/)

[OCI API Endpoints](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apiref.htm)

[Request Signature Implementation](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/signingrequests.htm)

# Setup

Import the collection and environment files into Postman

First run 'LOAD JSRSASign' once to load in the RSA-SHA256 signing library then you are ready to run the next calls. They are meant to be ran sequentially. You should also be able to run the complete scope of OCI API calls, but I haven't tested them all.

Don't forget to set the variables in the OCI environment

# OCI Environment
**tenancyId**: OCID of your tenancy

**userId**: OCID of the user

**fingerprint**: fingerprint of user's API Key

**namespaceName**: namespace of the tenancy

**compartmentId**: compartment to create the bucket

**secret**: entire PEM key of the user. For macOS run 'pbcopy < ~/.oci/oci_api_key.pem' assuming you have set up the keys for OCI previously

**passphrase**: optional. passphrase for the private key if one was set

**endpoint**: optional. endpoint of the API call. see https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apiref.html

### Don't have a secret key or fingerprint?
Visit this link [here](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apisigningkey.htm#How). **_Don't forget to upload the public key to your user account! This is necessary to make API calls_**. 

# Tutorial
To run a request, just press the big blue Send button on the top right.

## Step 1: Load JSRSASign
Before you can make any API calls, you must run the `Load JSRSASign` request. This will retrieve the javascript library [JSRSASign](http://kjur.github.io/jsrsasign/) and set code to be a global variable in Postman, which will then be initialized using the javascript eval() function. Why do we need this library? The OCI REST API authorization header requires us to be able to do signing with RSA. The CryptoJS library within Postman only supports signing with HMAC, so unfortunately we must use this workaround. The good news is that we only have to run this request once, and then we can make as many requests as we want until we close Postman. 

## Step 2: POST CreateBucket
This request will create an Object Storage bucket called `postman_bucket` in the compartmentId you specified in the `us-phoenix-1` region. Let's break down this API call. Because we are using the Object Storage Service API in Phoenix, we need to pick the correct endpoint in the documentation [here](https://docs.cloud.oracle.com/iaas/api/#/en/objectstorage/20160918/). We check the list of endpoints and decide on `https://objectstorage.us-phoenix-1.oraclecloud.com`. 

Now we must append a request target to the endpoint. Since we are creating a bucket, we will check the [CreateBucket documentation](https://docs.cloud.oracle.com/iaas/api/#/en/objectstorage/20160918/Bucket/CreateBucket) and see that the request target is `/n/{namespaceName}/b/`. Append that to our endpoint to create the request url `https://objectstorage.us-phoenix-1.oraclecloud.com/n/{namespaceName}/b/`. 

Let's take a look at the **Parameters** section of the CreateBucket page. The parameter `namespaceName` is a required parameter that is inserted in the _path_ of the request target (not to be mistaken as a query parameter!). Thankfully, our environment variables that we configured earlier take care of that for us.

Scroll further down to the **Body** section of the page. Since this is a POST request, a body is required in the request. The documentation says the body must be a [CreateBucketDetails resource](https://docs.cloud.oracle.com/iaas/api/#/en/objectstorage/20160918/datatypes/CreateBucketDetails). This basically means to construct a JSON object with the keys and values described in the CreateBucketDetails page. Since we only need the `name` and `compartmentId` attribute, our CreateBucketDetails resource looks like this
```json
{"compartmentId":"{{compartmentId}}", "name":"postman_bucket"}
```
Notice that we are using our Postman environment variables for the compartmentId value! Aren't they cool? Anyway, we add this body to the body tab in our Postman request, making sure that we are in the raw mode and our content type is application/json. 
 
When we press Send, our request should look like this
 
 ```
POST /n/<namespaceName>/b/
accept: */*
accept-encoding: gzip, deflate
authorization: <constructed in pre-request script. ignore for now>
cache-control: no-cache
content-length: 93
content-type: application/json
date: Wed, 20 Feb 2019 16:59:30 GMT
host: objectstorage.us-phoenix-1.oraclecloud.com
postman-token: 2bd9e479-239e-4bee-aac7-00b5f67b1d2f
user-agent: PostmanRuntime/7.3.0
x-content-sha256: LMy5rlZ2eXwSTk/LbKujb+VcqvJYgZ48dyr6yBfiJnM=
{"compartmentId":"ocid.compartment.oc1..exampleuniquecompartmentID", "name":"postman_bucket"}
 ```
There is an extra header that our pre-request script constructed called `x-content-sha256`. This isn't necessary to understand now, but just know that all requests that have a body (POST and PUT) need that body to be hashed with SHA256 and added as a header. Other required headers for POST and PUT requests are content-type, content-length, date, host, and authorization.

Phew, that was a lot! Now we should know how to navigate the OCI documentation to find endpoints and request targets and create our request URL, add path parameters to the URL, and construct a body for the POST request. The important thing is getting a feel for where in the documentation you can expect to find the information you need. If you understand this, then everything else is easy!

## Step 3: GET ListBuckets

GET requests (and DELETE requests) are simpler than POST and PUT requests. They don't have a body and only require date, host, and authorization headers (which are taken care of in the pre-request script). Can you construct the URL yourself before check the request in Postman? Start at the [OCI API Reference](https://docs.cloud.oracle.com/iaas/api/#/)!

When you're ready, hit Send and you should receive a list of [Bucket resources](https://docs.cloud.oracle.com/iaas/api/#/en/objectstorage/20160918/Bucket/), one being the resource for `postman_bucket`.

## Step 4: PUT PutObject
This request will add a text file in the bucket `postman_bucket` that reads 'Hello world!'. Now usually, constructing PUT requests are nearly identical to POST requests with the EXCEPTION being [PutObject](https://docs.cloud.oracle.com/iaas/api/#/en/s3objectstorage/20160918/Object/PutObject) and [UploadPart](https://docs.cloud.oracle.com/iaas/api/#/en/s3objectstorage/20160918/Object/UploadPart). These are different because they don't need the content of their body to be hashed, therefore making uploads more performant. The PutObject request body must be the binary string of the object to upload, and PutObject doesn't even have a body. I did not program the pre-request script  to handle these special cases because I figured Postman shouldn't be used to upload a non-trivial amount of data. You can read a bit more about the special cases [here](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/signingrequests.htm#ObjectStoragePut), but this page mainly discusses how to construct a signing string.

In the body tab in Postman, the body should contains the string `Hello world!`. The body should be in raw mode and the content type should be text/plain. Now press Send, and try to download the file `postman_object.txt` that should be in `postman_bucket`!

## Step 5 & 6: DELETE DeleteObject & DELETE DeleteBucket
The last two requests will delete the object `postman_object.txt` and then delete the bucket `postman_bucket`. DELETE requests are much like GET requests, so you should try constructing the DELETE requests yourself! If you're short for time or still learning, just press Send on DeleteObject and DeleteBucket and that'll be the end of the tutorial!

All these requests were made through the Object Storage Service API, but you should be able to now use any other services listed in the API References page, provided that you change your endpoint and construct the request correctly. Of course, I wasn't able to test every single service and request, so let me know if you run into any issues.

# How to create an authorization header (or what the heck does the pre-request script do?)
This section is for people who want to _really_ understand the OCI REST API and maybe learn how to make API calls somewhere besides Postman. The OCI REST API follows the HTTP Signature protocal ([draft-cavage-http-signatures-08](https://tools.ietf.org/html/draft-cavage-http-signatures-08)) for authorization. This requires the constructing many elements that could be a little confusing for someone who just started trying to learn the OCI API. I will attempt to distill them into simple, easy to understand tasks. This is basically a summary of the [Request Signatures documentation](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/signingrequests.htm).

## Step 1: Create a signing string

A signing string is a multi-line string that contains a single header and it's value in each line.

GET and DELETE requests need the headers `(request-target)`, `(host)`, and `date`.
PUT AND POST requests need the headers `(request-target)`, `(host)`, `date`, `x-content-sha256`, `content-type`, and `content-length`.

What are we going to do with this signing string? We are going to hash it with SHA256, and then we are going to sign it with a RSA private key.

Here is an example of a signing string for a POST request. Please note at the end of each line is the `\n` character which signifies a new line.

```
date: Thu, 05 Jan 2014 21:31:40 GMT
(request-target): post /20160918/volumeAttachments
host: iaas.us-phoenix-1.oraclecloud.com
content-length: 316
content-type: application/json
x-content-sha256: V9Z20UJTvkvpJ50flBzKE32+6m2zJjweHpDMX/U4Uy0=
```
The headers can be in any order, but you must remember that order for later!

### Explanation of each header
**date**: The date the request was made in UTC or GMT time.

**(request-target)**: The request method along with the URL snippet that comes after the endpoint/host portion.

**content-length**: The length of the body, which should be a string (even if it is JSON).

**content-type**: The content-type of the body.

**x-content-sha256**: The content of the body hashed with SHA256.

## Step 2: Hash the signing string

Now we will use the SHA256 algorithm to hash the string. If I were using Python with the Pycryptodome library, it would look like this

```python
>>> from Crypto.Hash import SHA256 

>>> signing_string = """date: Thu, 05 Jan 2014 21:31:40 GMT
(request-target): post /20160918/volumeAttachments
host: iaas.us-phoenix-1.oraclecloud.com
content-length: 316
content-type: application/json
x-content-sha256: V9Z20UJTvkvpJ50flBzKE32+6m2zJjweHpDMX/U4Uy0="""

>>> hashed_signing_string = SHA256.new((bytes(signing_string, 'utf-8'))) #returns a hash object
>>> print(hashed_signing_string.hexdigest()) #prints the hexadecimal string of the hash
'b0767ea900475d106b82f51642141749e0872a1236ab48219ab3d8123f4eac89'
```

## Step 3: Sign the hashed signing string and then encode to base64

Now we will use our private RSA key to sign the hashed string then encode to base64. If I were using Python, it would look like this

```python
>>> from Crypto.PublicKey import RSA
>>> from Crypto.Signature import PKCS1_v1_5
>>> from base64 import b64encode

>>> private_key = """-----BEGIN RSA PRIVATE KEY-----
MIICXgIBAAKBgQDCFENGw33yGihy92pDjZQhl0C36rPJj+CvfSC8+q28hxA161QF
NUd13wuCTUcq0Qd2qsBe/2hFyc2DCJJg0h1L78+6Z4UMR7EOcpfdUE9Hf3m/hs+F
UR45uBJeDK1HSFHD8bHKD6kv8FPGfJTotc+2xjJwoYi+1hqp1fIekaxsyQIDAQAB
AoGBAJR8ZkCUvx5kzv+utdl7T5MnordT1TvoXXJGXK7ZZ+UuvMNUCdN2QPc4sBiA
QWvLw1cSKt5DsKZ8UETpYPy8pPYnnDEz2dDYiaew9+xEpubyeW2oH4Zx71wqBtOK
kqwrXa/pzdpiucRRjk6vE6YY7EBBs/g7uanVpGibOVAEsqH1AkEA7DkjVH28WDUg
f1nqvfn2Kj6CT7nIcE3jGJsZZ7zlZmBmHFDONMLUrXR/Zm3pR5m0tCmBqa5RK95u
412jt1dPIwJBANJT3v8pnkth48bQo/fKel6uEYyboRtA5/uHuHkZ6FQF7OUkGogc
mSJluOdc5t6hI1VsLn0QZEjQZMEOWr+wKSMCQQCC4kXJEsHAve77oP6HtG/IiEn7
kpyUXRNvFsDE0czpJJBvL/aRFUJxuRK91jhjC68sA7NsKMGg5OXb5I5Jj36xAkEA
gIT7aFOYBFwGgQAQkWNKLvySgKbAZRTeLBacpHMuQdl1DfdntvAyqpAZ0lY0RKmW
G6aFKaqQfOXKCyWoUiVknQJAXrlgySFci/2ueKlIE1QqIiLSZ8V8OlpFLRnb1pzI
7U1yQXnTAEFYM560yJlzUpOb1V4cScGd365tiSMvxLOvTA==
-----END RSA PRIVATE KEY-----"""

>>> rsakey = RSA.importKey(private_key) # create RSAKey object
>>> signer = PKCS1_v1_5.new(rsakey) # use the PKCS1v15 padding scheme
>>> signed_hashed_signing_string = b64encode(signer.sign(digest)) # sign the hash object and then encode it to base64

>>> print(signed_hased_signing_string) 
b'Mje8vIDPlwIHmD/cTDwRxE7HaAvBg16JnVcsuqaNRim23fFPgQfLoOOxae6WqKb1uPjYEl0qIdazWaBy/Ml8DRhqlocMwoSXv0fbukP8J5N80LCmzT/FFBvIvTB91XuXI3hYfP9Zt1l7S6ieVadHUfqBedWH0itrtPJBgKmrWso='
```
## Step 4: Construct a keyId string
