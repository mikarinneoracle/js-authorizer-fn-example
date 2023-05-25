# API Gateway authorizer function context var example

### Build and deploy the functions

#### Authorizer function fnauthjs
<pre>
const fdk=require('@fnproject/fdk');

fdk.handle(function(input){
  let json = "";

  if(input.token) {
      json = {
        "active": true,
        "principal": "myprincipal",
        "scope": ["fnsimplejs"],
        "clientId": "clientIdFromHeader",
        "expiresAt": "2023-12-31T00:00:00+00:00",
        "context": {
            "username": input.token
        }
      }
  } else {
      json = {
        "active": false,
        "expiresAt": "2023-12-31T00:00:00+00:00",
        "wwwAuthenticate": "Bearer realm=\"www.com\""
      }
  }
  return json;
})
</pre>

The authorizer function will pass on the <code>username</code> in <code>auth context</code> as a custom variable. The value for it is set from REST call input as <code>token</code> i.e.
<pre>
curl -H "token: test-token" https://drp....56kvgu.apigateway.eu-amsterdam-1.oci.customer-oci.com/
</pre>
Hence the auth context var <code>username</code> gets the value <code>test-token</code>

<p>
If no <code>token</code> is given in the REST call the function will return <b>401 Unauthorized</b> error e.g. 
<pre>
curl https://drp....56kvgu.apigateway.eu-amsterdam-1.oci.customer-oci.com/
{"code":401,"message":"Unauthorized"}
</pre>

#### Backend / secondary function fnsimplejs
<pre>
const fdk=require('@fnproject/fdk');

fdk.handle(function(input, ctx){
  return ctx.headers['Fn-Http-H-Username'];
})    
</pre>

The secondary / backend function will get the authorizer passed variable <code>username</code>
as a transformed header variable as <code>Fn-Http-H-Username</code> and will print it out as the
function REST call result i.e.

<pre>
curl -H "token: test-token"  https://drp....56kvgu.apigateway.eu-amsterdam-1.oci.customer-oci.com/
["test-token"]
</pre>

### Create the API Gateway based on the functions and configure as follows

#### Authorizer function fnauthjs

<img src="./authorizer-function.png" width="800" />
<p>
 
#### Backend / secondary function fnsimplejs

<img src="./backend-function.png" width="800" />
<p>
    
#### <i>auth context</i> variable <i>username</i> transformation to a header

<img src="./header-transformations.png" width="800" />
<p>
