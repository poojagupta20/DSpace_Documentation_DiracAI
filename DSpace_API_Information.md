
# DSpace API information

## Default DSpace APIs


### Login APIs and login process
  -- The first step is to do GET request to status API to get the CSRF token `http://localhost:8080/server/api/authn/status`
  -- The json return  object for this API: 
     `{
    "id": null,
    "okay": true,
    "authenticated": false,
    "authenticationMethod": null,
    "type": "status",
    "_links": {
        "eperson": {
            "href": "http://localhost:8080/server/api/authn/status/eperson"
        },
        "specialGroups": {
            "href": "http://localhost:8080/server/api/authn/status/specialGroups"
        },
        "self": {
            "href": "http://localhost:8080/server/api/authn/status"
        }
    }`

    -- In the response object 

    -- Make a POST request to `http://localhost:8080/server/api/authn/login` with the following json body. json body will contain the CSRF token generated from the previous step and username & password.







    curl -X POST "http://localhost:8080/server/api/authn/login" \
     -H "Content-Type: application/x-www-form-urlencoded" \
     -H "X-XSRF-TOKEN: d5a6f713-3d20-464b-92f8-e710749b652c" \
     -b cookies.txt \
     -D headers.txt \
     --data-urlencode "user=admin@gmail.com" \
     --data-urlencode "password=admin"



### Community , Collection and Item view APIs




