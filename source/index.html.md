---
title: Slant API Documentation

language_tabs: # must be one of https://git.io/vQNgJ
- shell: cURL

toc_footers:
  - <a href='#'>Sign Up for a Developer Key</a>

includes:
  - errors

search: true
---

# Introduction

Welcome to the Slant API!  This API is organized around REST, and you can use it to access all client- and partner-related data.

> Important: For the sake of brevity, the `Content-Type` header will be omitted from this documentation. However, since JSON is used throughout the API, all requests must contain this header:

```shell
-H "Content-Type: application/json"
```


HTTPS must be used for all endpoints, and any unsecured (HTTP) requests will be rejected. Every endpoint in the API accepts and returns JSON, including errors. You can view code examples in the area to the right.

# Authentication

> Request authentication - applies to all API endpoints:

```shell
curl "https://api.slantreviews.com/v2/example"
    -H "Authorization: YOUR_TOKEN_HERE"
```

Slant uses JSON Web Tokens (JWT) to allow access to the API. To authenticate a request, you simply provide an access token in the request's `Authorization` header.

### Base URL

`https://api.slantreviews.com/v2`

### Request Headers

Header | Description
--------- | -----------
Authorization | Your access token. You must provide a token for all requests, unless otherwise noted.

There are two types of accounts that may be authenticated: *service accounts* and *user accounts*. The differences are described below.

## Service accounts

> Note: No login is needed for service accounts.

Service accounts are used by Slant partners and white-label customers to access certain endpoints with elevated permissions. These endpoints allow Slant partners to register new businesses, create users, view statistics, and perform other administrative tasks.

No login is necessary for service accounts. You are provided with a token when you register with Slant, and these tokens do not expire unless revoked.

**Note:** Service account tokens are intended for server-side use only, and should be guarded carefully. These tokens should never be shared, uploaded to public repositories like GitHub, or exposed to client-facing applications!

### Obtaining a user token

Service accounts always have the ability to make requests on behalf of a user, but sometimes it is necessary to to make requests directly from a client-facing application.  If this is the case, you should first consider whether it is possible to authenticate the user using their email+password as described under **User accounts**. If so, this is the preferred method. This is how the Slant web portal performs authentication, and this is how most standalone applications should work.

However, there are situations where it may not be possible to ask the user to log in with a password. An example of this is when you are integrating specific Slant features into an existing web application, such as a CRM. In this case you probably perform your own authentication, and the end user may not even be aware of Slant's existence.

In this case, you should retrieve a user token as shown below. The initial retrieval will be performed server-side by your service account, and then the token may be shared with the client to make requests directly to Slant's API.

> To retrieve a user token using a service account token:

```shell
curl "https://api.slantreviews.com/v2/users/:user_id/token"
    -H "Authorization: SERVICE_ACCOUNT_TOKEN_HERE"
```

> Example response:

```json
{
    "access_token": "12345678.aaaaaaaaa.0000000"
}
```

### HTTP request

`GET https://api.slantreviews.com/v2/users/:user_id/token`

### Request Parameters

Parameter | Description
--------- | -----------
user_id | The user's ID.

### Request Headers

Header | Description
--------- | -----------
Authorization | Your service account token.

<aside class="warning">If you believe that your service account token may have been compromised or exposed to the public, please contact Slant immediately to reset your token!</aside>


## User accounts

As the name suggests, user accounts are used by individual end users. These may be business owners, employees, or other people who have been given access to a company's Slant account (or white label equivalent).

A user may obtain a token by providing an email + password combo the the API, which will then return an access token if the credentials are valid.

### HTTP request

> To log in a user using email and password:

```shell
curl "https://api.slantreviews.com/v2/auth/login"
    -X POST
    -d '{"email": "USER_EMAIL", "password": "USER_PASSWORD"}'
```

> Example response:

```json
{
    "access_token": "12345678.aaaaaaaaa.0000000"
}
```

`POST https://api.slantreviews.com/v2/auth/login`

### Body Parameters

Parameter | Description
--------- | -----------
email | The user's email address.
password | The password provided by the user.

<aside class="success">This is the "standard" login method used by Slant. If you are building your own application, you should use this when possible.</aside>


# Users

## Get the current user

```shell
curl "https://api.slantreviews.com/v2/users/me"
  -H "Authorization: USER_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "id": "00000000-0000-0000-0000-000000000000",
    "first_name": "Bob",
    "last_name": "User",
    "email": "bob@example.com",
    "locations": [
        {
            "id": "00000000-0000-0000-0000-000000000000",
            "company_id": "00000000-0000-0000-0000-000000000000",
            "name": "Nom Nom - Center Street",
            "phone": "+18015556666",
            "client_role": "admin",
            "address": {
                "state": "UT",
                "city": "Provo",
                "address_1": "123 Center Street",
                "address_2": "Ste 2",
                "zip": "84604"
            }
        }
    ]
}
```

This endpoint retrieves details for the currently-authenticated user.

For most applications, this is the first endpoint that should called after a successful login. It will return basic information about the authenticated user, as well as a list of locations that the user is authorized to access.

Accessible only to user accounts.

### HTTP request

`GET https://api.slantreviews.com/v2/users/me`

### Request headers

Parameter | Description
--------- | -----------
Authorization | The user's access token.

## Get a specific user

```shell
curl "https://api.slantreviews.com/v2/users/:user_id"
  -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "id": "00000000-0000-0000-0000-000000000000",
    "first_name": "Bob",
    "last_name": "User",
    "email": "bob@example.com",
    "locations": [
        {
            "id": "00000000-0000-0000-0000-000000000000",
            "company_id": "00000000-0000-0000-0000-000000000000",
            "name": "Nom Nom - Center Street",
            "phone": "+18015556666",
            "client_role": "admin",
            "address": {
                "state": "UT",
                "city": "Provo",
                "address_1": "123 Center Street",
                "address_2": "Ste 2",
                "zip": "84604"
            }
        }
    ]
}
```

This endpoint retrieves a specific user.

This endpoint is accessible to user accounts and service accounts.

### HTTP request

`GET https://api.slantreviews.com/v2/users/:user_id`

### Request Parameters

Parameter | Description
--------- | -----------
user_id | The user's ID.


