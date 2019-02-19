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
