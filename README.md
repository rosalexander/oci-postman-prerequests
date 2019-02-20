# Helpful Links
[OCI API Reference](https://docs.cloud.oracle.com/iaas/api/#/)

[OCI API Endpoints](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/apiref.htm)

[Request Signature Implementation](https://docs.cloud.oracle.com/iaas/Content/API/Concepts/signingrequests.htm)

# Setup

Import the collection and environment files into Postman

First run 'LOAD JSRSASign' once to load in the RSA-SHA256 signing librarym then you are ready to run the next calls. They are meant to be ran sequentially. You should also be able to run the complete scope of OCI API calls, but I haven't tested them all.

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

# Running the collection
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
