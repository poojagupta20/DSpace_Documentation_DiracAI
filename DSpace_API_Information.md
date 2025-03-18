
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


### Community , Collection and Item view APIs




