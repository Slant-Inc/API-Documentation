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

### Request headers

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

### Request parameters

Parameter | Description
--------- | -----------
user_id | The user's ID.

### Request headers

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

### Body parameters

Parameter | Description
--------- | -----------
email | The user's email address.
password | The password provided by the user.

<aside class="success">This is the "standard" login method used by Slant. If you are building your own application, you should use this when possible.</aside>

## User account confirmation

> To confirm a user account and set a new password:

```shell
curl "https://api.slantreviews.com/v2/auth/reset/:reset_code"
    -X POST
    -d '{"new_password": "USER_PASSWORD"}'
```

> Example response:

```json
{
    "access_token": "12345678.aaaaaaaaa.0000000"
}
```

When a new user account is created or a password reset is requested, the user is sent a confirmation email containing a unique link.  This link will direct the user to a page where they can create a password and sign in.

White-label partners must build a page to handle this functionality. The link provided to the user will be formatted like this:

`https://{domain}/reset/{reset_code}?email={user_email}`

where `domain` is the app domain provided when you signed up as a Slant partner and `user_email`  is the user's email address.  For example, native Slant users receive an email containing a link to this address:

`https://app.slantreviews.com/reset/{reset_code}?email=example@example.com`

where `reset_code` is a unique code used to verify the user's identity.

If you are developing a white-label confirmation page, this page should provide a field that asks the user to set a password. Once the user submits the password, the page should send a request to the following endpoint:

### HTTP request

`POST https://api.slantreviews.com/v2/auth/reset/:reset_code`

### Request parameters

Parameter | Description
--------- | -----------
reset_code | The unique reset code from the user's link. This will need to be extracted from the URL in the address bar.

### Body parameters

Parameter | Description
--------- | -----------
new_password | The new password provided by the user.

If a valid reset code and an acceptable password are provided, this endpoint will return an access token. The user may then be directed to the application as they normally would after a successful login.

## Requesting a password reset (non-authenticated)

> To request a password reset link via email:

```shell
    curl "https://api.slantreviews.com/v2/auth/reset_link"
    -X POST
    -d '{"email": "USER_EMAIL", "domain": "APP_DOMAIN"}'
```

> Example response:

```json
{
    "message": "Password reset link sent."
}
```

If a user forgets their password, they may request a password reset by confirming their email address.  A unique link is then sent to their inbox which allows them to set a new password.

Since the user is not authenticated during this process, this endpoint requires the `domain` to be sent along with the user's email address.  This allows the Slant API to identify when the user is sending the request from a white-label application, and maintain consistent branding in the email that is sent.

For white-label appliations, the `domain` should be extracted from the page's URL.  For example, if the user is visiting `https://app.slantreviews.com/login` and wishes to reset their password, the `domain` would be `app.slantreviews.com`.

Once the user receives the unique confirmation link, the remainder of the process is identical to when their account was first created.  See [User account confirmation](#user-account-confirmation).

**Important:** This endpoint should only be used for *non-authenticated* password resets (i.e. when a user forgets their password).  To reset a user's password while currently signed in, see [Resetting a password for a user (authenticated)](#resetting-a-password-for-a-user-authenticated).

### HTTP request

`POST https://api.slantreviews.com/v2/auth/reset_link`

### Body parameters

Parameter | Description
--------- | -----------
email | The email address provided by the user.
domain | The web domain from which the user is requesting a password reset.

## Resetting a password for a user (authenticated)

> To reset an authenticated user's password:

```shell
curl "https://api.slantreviews.com/v2/users/:user_id/password"
    -X POST
    -d '{"current_password": "USER_CURRENT_PASSWORD", "new_password": "USER_NEW_PASSWORD"}'
```

> Example response:

```json
{
    "access_token": "12345678.aaaaaaaaa.0000000"
}
```

A user may choose to reset their password at any time while signed in.  They must provide their current password and a new password, with a minimum length of 6 characters.

### HTTP request

`POST https://api.slantreviews.com/v2/users/:user_id/password`

### Request parameters

Parameter | Description
--------- | -----------
user_id | The user's ID.

### Body parameters

Parameter | Description
--------- | -----------
current_password | The user's current password.
new_password | The new password provided by the user.


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

> To fetch a specific user:

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

### Request parameters

Parameter | Description
--------- | -----------
user_id | The user's ID.

## Generate a user access token

> To generate a user access token:

```shell
curl "https://api.slantreviews.com/v2/users/:user_id/token"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "access_token": "12345678.aaaaaaaaa.0000000"
}
```

This endpoint allows partner service accounts to retrieve a user access token on behalf of a user.  This would generally be used for API-only partners, where it may be necessary to perform "user" actions (such as sending review invitations) from a custom application.

Accessible only to partner service accounts.

### HTTP request

`GET https://api.slantreviews.com/v2/users/:user_id/token`

### Request parameters

Parameter | Description
--------- | -----------
user_id | The user's ID.


# Companies

In Slant, a `Company` represents a business entity which contains one or more [Locations](#locations).  This heirarchy makes it easy to represent multiple real-world business locations that operate under the same brand.

Only partner service accounts may create, modify or delete companies.

## Get all companies

> To fetch a complete list of companies:

```shell
curl "https://api.slantreviews.com/v2/companies"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "companies": [
        {
            "id": "00000000-0000-0000-0000-000000000000",
            "name": "Nom Nom Foods",
            "date_created": 1507201073.95566,
            "notes": "Example company 1"
        },
        {
            "id": "00000000-0000-0000-0000-000000000001",
            "name": "Sunnyside Dental",
            "date_created": 1507201073.95566,
            "notes": "Example company 2"
        }
    ]
}
```

This endpoint retrieves a list of all companies "owned" by a partner service account (i.e. all companies that have been created by this account).

Accessible only to service accounts.

### HTTP request

`GET https://'api.slantreviews.com/v2/companies`

## Create a company

> To create a new company:

```shell
curl "https://api.slantreviews.com/v2/companies"
    -X POST
    -d '{"name": "Nom Nom Foods", "notes": "Example account."}'
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> If successful, the above command returns JSON structured like this:

```json
{
    "id": "00000000-0000-0000-0000-000000000000",
    "name": "Nome Nom Foods",
    "notes": "Example account.",
    "date_created": 1515797239.05312
}
```

This endpoint creates a new company account. It should be used for registering new businesses with Slant.

Accessible only to service accounts.

<aside class="notice">After creating a company, a location must also be created (using the new company's ID) in order for the business to start using Slant.</aside>

### HTTP request

`POST https://api.slantreviews.com/v2/companies`

### Body parameters

Parameter | Description
--------- | -----------
name | The company's name.
notes | (Optional) Any notes to attach to the company's profile (only visible to the partner service account).


# Locations

In Slant, a `Location` represents a single physical business location. Every location operates under a parent [Company](#companies).

If a business contains more than one physical storefront, it is recommended that each storefront be represented in Slant as its own Location. This is a rule imposed by many review platforms, so following the same pattern in Slant ensures that our system can collect reviews accurately.

Only partner service accounts may create, modify or delete locations.

## Get all locations for a company

> To fetch all locations under a specific company:

```shell
curl "https://api.slantreviews.com/v2/companies/:company_id/locations"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "locations": [
        {
            "id": "00000000-0000-0000-0000-000000000000",
            "company_id": "00000000-0000-0000-0000-000000000001",
            "name": "Nom Nom Foods",
            "address": {
                "address_1": "123 Sesame Street",
                "address_2": "Ste 101",
                "city": "Provo",
                "state": "UT",
                "zip": "84604"
            },
            "phone": "+12223334444"
        }
    ]
}
```

This endpoint retrieves a list of all locations under a specific company.

Accessible only to partner service accounts.

### HTTP request

`GET https://api.slantreviews.com/v2/companies/:company_id/locations`

### Request parameters

Parameter | Description
--------- | -----------
company_id | The company's ID.

## Create a location

> To create a new location:

```shell
curl "https://api.slantreviews.com/v2/companies/:company_id/locations"
    -X POST
    -d '{"name": "Nom Nom Provo", "address_1": "123 Sesame Street", "address_2": "Ste 101", "city": "Provo", "state": "UT", "zip": "84604", "phone": "+12223334444", "notes": "Example account."}'
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> If successful, the above command returns JSON structured like this:

```json
{
    "id": "00000000-0000-0000-0000-000000000000",
    "company_id": "00000000-0000-0000-0000-000000000001",
    "name": "Nom Nom Provo",
    "address": {
        "address_1": "123 Sesame Street",
        "address_2": "Ste 101",
        "city": "Provo",
        "state": "UT",
        "zip": "84604"
    },
    "phone": "+12223334444"
}
```

This endpoint creates a new location under a specific company.  Note that a company must be created *before* the location.

Accessible only to partner service accounts.

<aside class="notice">The process for creating a location is somewhat extensive, and includes the registration of a new phone number with Slant's SMS provider. For this reason, requests may take up to 15 seconds to complete.</aside>

### HTTP request

`POST https://api.slantreviews.com/v2/companies/:company_id/locations`

### Request parameters

Parameter | Description
--------- | -----------
company_id | The ID of the parent company under which to create the location.

### Body parameters

Parameter | Description
--------- | -----------
name | The name of the location. May be the same as the company name, or unique.
address_1 | The location's address line 1.
address_2 | (Optional) The location's address line 2.
city | The location's city.
state | The location's 2-character state code (e.g. UT).
zip | The location's zip code (as a string).
phone | The location's business phone number. Any format with an area code is acceptable.
notes | (Optional) Any notes to attach to the location's profile (only visible to the partner service account).


# Stats

## Get review stats for a location

> To fetch review stats:

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

`GET https://api.slantreviews.com/v2/locations/:location_id/stats`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.


# Feedback

## Get private feedback for a location

> To fetch all feedback:

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/feedback"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint retrieves an array of all private feedback for a location.

Accessible to user accounts where the user is authorized as a member or admin of this location. Accessible to service accounts.

### HTTP request

`GET https://api.slantreviews.com/v2/locations/:location_id/feedback`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.


# Reviews

## Get reviews for a location

> To fetch all reviews:

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

`GET https://api.slantreviews.com/v2/locations/:location_id/reviews`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.


# Review Accounts

## Get linked review accounts for a location

> To fetch all linked review accounts:

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/review_accounts"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "available_sites": [
        {
            "id": "facebook",
            "color": "3b5998",
            "name": "Facebook",
            "icon_url": "https://facebook_icon_url"
        },
        {
            "id": "google",
            "color": "3cba54",
            "name": "Google",
            "icon_url": "https://google_icon_url"
        },
        {
            "id": "yelp",
            "color": "c41200",
            "name": "Yelp",
            "icon_url": "https://yelp_icon_url"
        }
    ],
    "linked_accounts": [
        {
            "site": {
                "id": "facebook",
                "color": "3b5998",
                "name": "Facebook",
                "icon_url": "https://facebook_icon_url"
            },
            "order": 2,
            "url": "https://www.facebook.com/business_name",
            "response_url": "https://www.facebook.com/business_name/reviews",
            "total_reviews": 14,
            "average_rating": 4.4
        },
        {
            "site": {
                "id": "google",
                "color": "3cba54",
                "name": "Google",
                "icon_url": "https://google_icon_url"
            },
            "order": 1,
            "url": "https://www.google.com/maps/place/business_address",
            "response_url": "https://www.google.com/search?q=business_address",
            "total_reviews": 8,
            "average_rating": 4.6
        },
    ]
}
```

This endpoint retrieves an array of all review accounts currently linked to a location, as well as a list of available sites that may be linked.

Accessible to user accounts where the user is authorized as an admin of this location. Accessible to service accounts.

### HTTP request

`GET https://api.slantreviews.com/v2/locations/:location_id/review_accounts`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.


## Link a new review account

> To link a new review account:

```shell
curl "https://api.slantreviews.com/v2/auth/login"
    -X POST
    -d '{"site_id": "REVIEW_SITE_ID", "url": "REVIEW_ACCOUNT_URL"}'
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> Note: If successful, the above command returns an updated list of review accounts for the location. See endpoint to GET review accounts above.

This endpoint allows a location to link a new review account for their business.

Linking is performed by selecting a valid site and submitting the URL from the location's review account on that site. Please note that the review site must be one of the `available_sites` returned  by `GET review accounts` above.

**Example:**  Nom Nom Foods has a Facebook review page located at `https://www.facebook.com/pg/nomnomfoodtrucks/reviews/`. In order to link this account to Slant, a POST would be submitted to this endpoint where the request body looks like this:

`{"site_id": "facebook", "url": "https://www.facebook.com/pg/nomnomfoodtrucks/reviews/"}`

**Note:** Upon successfully linking an account, this endpoint returns an updated list of *all* review accounts linked to this location, as well as an updated list of available sites.

Accessible to user accounts where the user is authorized as an admin of this location. Accessible to service accounts.

### HTTP request

`POST https://api.slantreviews.com/v2/locations/:location_id/review_accounts`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.

### Body parameters

Parameter | Description
--------- | -----------
site_id | The ID of the review site being linked. This must be an ID from one of the available sites returned by `GET review accounts`.
url | The URL pointing to the location's review page on this site.

## Delete a review account

```shell
    curl "https://api.slantreviews.com/v2/locations/:location_id/review_accounts/:site_id"
    -X DELETE
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> Note: If successful, the above command returns an updated list of review accounts for the location. See endpoint to GET review accounts above.

This endpoint allows a location to un-link a review account from Slant.

**Note:** After successfully un-linking an account, this endpoint returns an updated list of *all* review accounts linked to this location, as well as an updated list of available sites.

### HTTP request

`DELETE https://api.slantreviews.com/v2/locations/:location_id/review_accounts/:site_id`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.
site_id | The ID of the review site to un-link. Must be a site from a review account currently linked to this location.


# Invitations

One of the primary functionalities of Slant is the ability to send review invitations. These invitations are sent to customers via SMS text message, and assist them with the process of leaving a review for the business.

## Get review invitations for a location

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/review_invitations"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "review_invitations": [
        {
            "id": "RFyQV0",
            "client_number_e164": "+15556667777",
            "client_number_natural": "(555) 666-7777",
            "client_name": "Bob",
            "date_sent": 1507201073.95566,
            "date_clicked": 1507201143.24624,
            "recommended": false,
            "rating_date": 1507201176.56359,
            "feedback_body": "This restaurant isn't very good. I will not be returning.",
            "feedback_date": 1507201264.92683
        },
        {
            "id": "GvwN58",
            "client_number_e164": "+16667778888",
            "client_number_natural": "(666) 777-8888",
            "client_name": "Joe"
            "date_sent": 1507201066.95727
        }
    ]
}
```

> Note: `"date_clicked"`, `"recommended"`, `"rating_date"`, `"feedback_body"` and `"feedback_date"` are all optional, and will only be returned if the customer has completed those steps in the review process.

This endpoint retrieves an array of all review invitations that have been sent by a location. This allows a business to review their customer history, and resend invitations if desired.

Accessible to user accounts and service accounts.

### HTTP request

`GET https://api.slantreviews.com/v2/locations/:location_id/review_invitations`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.

## Send a review invitation

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/review_invitations"
    -X POST
    -d '{"to_number": "PHONE_NUMBER", "to_name": "CUSTOMER_NAME"}'
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint sends a new review invitation to a customer.

When sending an invitation, the customer's phone number must be included but their name is optional.  Phone numbers must be a minimum of 10 digits (including area code). If a country code is not included, it is assumed to be a United States number (+1). Any format is acceptable, as all non-numerical characters will be removed.

### HTTP request

`POST https://api.slantreviews.com/v2/locations/:location_id/review_invitations`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.

### Body parameters

Parameter | Description
--------- | -----------
to_number | The customer's phone number. Minimum 10 digits, any format.
to_name | (Optional) The customer's first name.


## Get the invitation message body

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/review_invitation_body"
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint allows a location to view the message body that is sent to customers via SMS.

There are two types of message body, which are detailed below. See [Modify the invitation message body](#modify-the-invitation-message-body).

### HTTP request

`GET https://api.slantreviews.com/v2/locations/:location_id/review_invitation_body`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.


## Modify the invitation message body

```shell
curl "https://api.slantreviews.com/v2/locations/:location_id/review_invitation_body"
    -X PATCH
    -d '{"primary": "PRIMARY_MESSAGE_BODY", "fallback": "FALLBACK_MESSAGE_BODY"}'
    -H "Authorization: ACCESS_TOKEN_HERE"
```

> The above command returns JSON structured like this:

```json
{
    "TODO": "DISPLAY EXAMPLE RESPONSE HERE"
}
```

This endpoint allows a location to customize the text message that is sent to customers via SMS.

There are two types of message body, referred to here as  **primary** and **fallback**:

### Primary message body

This is the message that will be sent to customers when a name is included in the request (recommended). An example is shown below:

`"Hi {{NAME}}, thanks for choosing XYZ Auto Repair! Do you mind taking a moment to leave us a review? This link makes it easy: {{LINK}}"`

Note the usage of mustache-style placeholders in this example body: `{{NAME}}` and `{{LINK}}`.  When an invitation is sent, these placeholders will be replaced with the customer's provided name and a unique link to leave a review.

<aside class="notice">It is important that the primary message body include BOTH mustache placeholders shown above. If they are not included, the message will be considered invalid and a default message body will be sent.</aside>

### Fallback message body

This is the message that will be sent when the customer's name is not provided.  An example is shown below:

`"Hi there! Thanks for choosing XYZ Auto Repair! Do you mind taking a moment to leave us a review? This link makes it easy: {{LINK}}"`

Note that the fallback message body does not include a `{{NAME}}` placeholder, but still includes `{{LINK}}`.

<aside class="notice">It is important that the fallback message body includes a link placeholder, but NOT a name placeholder. If it does not follow this format, the message will be considered invalid and a default message body will be sent.</aside>

### HTTP request

`PATCH https://api.slantreviews.com/v2/locations/:location_id/review_invitation_body`

### Request parameters

Parameter | Description
--------- | -----------
location_id | The location's ID.

### Body parameters

Parameter | Description
--------- | -----------
primary | (Optional) The new primary message body.
fallback | (Optional) The new fallback message body.

Accessible to user accounts where the user is authorized as an admin of this location. Accessible to service accounts.

<aside class="success">This PATCH endpoint allows one or both message bodies to be updated in a single request. If one is omitted, the existing value will be maintained.</aside>

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

### Body parameters

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

