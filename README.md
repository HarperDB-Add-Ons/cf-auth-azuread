# HarperDB Azure Active Directory Authentication
This package is an Active Directory authentication validator for a HaperDB Custom Function.

This adds the ability to secure database access with an Azure Active Directory application to validate against AD Users and Roles.
## How To Use
1. Create an Azure Active Directory application with the desired users and roles.
2. Include the ClientID, AuthorityURL, and ClientSecret as environment variables.
3. Include this package in the routes defition file (i.e. `$CUSTOM_FUNCTION_DIR/routes/index.js`)
4. Add the validate function to the prevalidation handler for endpoints that should use AAD Authentication.

## Environment Variables
- AAD_CLIENT_ID="the client id for the AAD app"
- AAD_AUTH_URL="the authority url for the AAD app"
- AAD_CLIENT_SECRET="the client secret for the AAD app"
- AAD_REDIRECT_URI="the URL to redirect after a successful validation"

## Example Route and POST
### Route Definition
```
// /routes/index.js

const aadAuth = require('harperdb-active-directory-auth')

module.exports = async (server, { hdbCore, logger }) => {
  // CREATE A DATA RECORD
  server.route({
    url: '/:schema/:table',
    preValidation: (request, response, next) => aadAuth.validate(request, response, next, hdbCore, logger),
    method: 'POST',
    handler: (request) => {
      const { schema, table } = request.params;
      const { records } = request.body;
      request.body = {
         operation: 'insert',
         schema,
         table,
         records,
         hdb_user: request.body.hdb_user,
      }

      return hdbCore.request(request)
    }
  })
})
```
### cURL POST
```
curl -X POST http://127.0.0.1:9926/cool-app/dev_schema/dogs_table \
   -H 'Content-Type: application/json' \
   -d '{
      "username":"dog@harperdb.io",
      "password":"D0G5RUL3!",
      "records":[{"name": "Bruno", "breed": "awesome"}]
   }'
   
```
