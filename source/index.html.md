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


# Roles

User accounts may be granted one of two roles when authorized for a location:

### Member

Members are granted access to basic location information including stats and reviews.  They cannot view or modify location settings such as review accounts, members or SMS settings.  Members may send review invitations.

This is the recommended role for most employees at a business.

### Admin

Admins are granted total administrative control over a location. They may view and modify all location settings, including review accounts, membership and SMS settings.

Business owners are automatically given admin access when they create a Slant account.  They may also choose to grant admin access to other managers or trusted employees.

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


# Companies

In Slant, a `Company` represents a business entity which contains one or more [Locations](#locations).  This heirarchy makes it easy to represent multiple real-world business locations that operate under the same brand.

Only partner service accounts may create, modify or delete companies.


# Locations

In Slant, a `Location` represents a single physical business location. Every location operates under a parent [Company](#companies).

If a business contains more than one physical storefront, it is recommended that each storefront be represented in Slant as its own Location. This is a rule imposed by many review platforms, so following the same pattern in Slant ensures that our system can collect reviews accurately.

Only partner service accounts may create, modify or delete locations.


# Stats

## Get review stats for a location

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/stats"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint retrieves basic review statistics for a specific location.

Accessible to user accounts where the user is authorized as a member or admin of this location. Accessible to service accounts.

### HTTP request

`GET https://'api.slantreviews.com/v2/locations/:location_id/stats`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.


# Reviews

## Get reviews for a location

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/reviews"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint retrieves an array of all online reviews for a location.

Accessible to user accounts where the user is authorized as a member or admin of this location. Accessible to service accounts.

### HTTP request

`GET https://'api.slantreviews.com/v2/locations/:location_id/reviews`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.


# Members

## Get members for a location

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/members"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint retrieves an array of all members and admins who are authorized for a specific location.

Accessible to user accounts where the user is authorized as an admin of this location. Accessible to service accounts.

### HTTP request

`GET https://'api.slantreviews.com/v2/locations/:location_id/members`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.

## Add a new member to a location

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/members"
    -X POST
    -d '{"first_name": "FIRST_NAME", "last_name": "LAST_NAME", "email": "EMAIL", "role": "ROLE"}'
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint authorizes a new user to be a member or admin of a location.

When this request is submitted, it is first determined whether a user with the provided email address already exists in the Slant system. If the user already has an account, then they are granted access to this location with the provided role.

If the user does not yet exist, a new account is created automatically and granted access to this location.  A confirmation email is sent to the provided address, which will allow the user to confirm their address, set a password and log in.

Changing a user account's own role is not allowed.

### HTTP request

`POST https://api.slantreviews.com/v2/locations/:location_id/members`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.

### Body Parameters

Parameter | Description
--------- | -----------
first_name | The user's first name.
last_name | (Optional) The user's last name.
email | The user's email address.
role | A valid role to assign the user. Either "admin" or "member". See [Roles](#roles).

<aside class="notice">This endpoint will also change a user's current role if needed.</aside>

## Delete a member from a location

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/members"
    -X DELETE
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint revokes a user's role (as a member or admin) from a location.

Attempting to revoke a user account's own role is not allowed.

### HTTP request

`DELETE https://api.slantreviews.com/v2/locations/:location_id/members/:user_id`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.
user_id | The ID of the user to remove.

<aside class="notice">This endpoint will not delete a user account entirely. It only revokes permission for the specified location.</aside>

