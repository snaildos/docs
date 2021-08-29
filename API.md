---
title: Official API for SnailDOS
description: Full API regarding SnailDOS
published: true
date: 2021-08-29T15:03:26.921Z
tags: development
editor: markdown
dateCreated: 2021-08-28T14:45:09.110Z
---

# Official Docs
Welcome to the docs!
=========================
------------------------------------------
00-responses
------------------------------

# General response

All responses follow a certain template that reports if the requested action was successful or not.

**All requests return a status property indicating if the action was successful or not**

# Successful actions

If an action is successful it follows this template under two conditions.

- If the requested action fetches a certain document from the database it returns the all the data that is accessible based on the user privileges

  - For example, fetching a user:

    ```json
    {
      "status": true,
      "user": {
        "id": "string",
        "createdAt": "number",
        "scopes": "string[]",
        "email": "string",
        "name": "string",
        "verified": "boolean"
      }
    }
    ```

  /!\ This has a single exception which is the "fetch currently playing" endpoint. (Refer to the proper section of the documentation for more)

- If the requested action modifies, deletes, to posts a certain document from the database For example, register user) it returns data that has been modified by consequence of the requested action or if the content was created it returns important data to access the newly created data

- For example, registering a new user:

  ```json
  {
    "status": true,
    "token": "string"
  }
  ```

# Failed actions

If an action has failed it follows this template under all conditions.

```jsonc
{
  "status": false,
  "message": "string"
  //...other properties may follow as details to what went wrong.
}
```

Failed actions provide an appropriate status code to what went wrong.

Some parts of the documentation may include status codes explaining the issues related to what went wrong

## Known failed actions responses

- **429**: Too many requests

  ```jsonc
  {
    "status": false,
    "global": true,
    "message": "too many requests",
    "try_again": "number" //Time in milliseconds
  }
  ```

  - This response signifies that the client has sent too much requests in a short sequence of time.

  ```jsonc
  {
    "status": false,
    "global": false,
    "message": "too many requests",
    "bucketKey": "string", //Unique bucket key generated to identify the action executed by the user
    "try_again": "number" //Time in milliseconds
  }
  ```

  - This response signifies that the client has already executed an action and needs to wait a certain amount of time before doing it again.

- **408**: Request time out

  ```jsonc
  {
    "status": false,
    "message": "request timeout"
  }
  ```

  - This response signifies that the client has took too long to deliver the body of the given request.

- **415**: Unsupported media type

  ```jsonc
  {
    "status": false,
    "message": "unsupported media type"
  }
  ```

  - This response signifies that the client has provided content that cannot be processed by the server.

- **400**: Invalid body provided

  ```jsonc
  {
    "status": false,
    "message": "invalid body provided"
  }
  ```

  - This response signifies that the client has corrupted JSON content that cannot be properly parsed by the server.

- **500**: Internal server error

  ```jsonc
  {
    "status": false,
    "message": "Internal server error"
  }
  ```

  - This response signifies that an unexpected error has been throw by one of the action handlers or routes.

- **404**: Endpoint was not found

  ```jsonc
  {
    "status": false,
    "method": "string",
    "url": "url",
    "message": "endpoint was not found"
  }
  ```

  - This response signifies that the requested endpoint or method cannot be processed as the appropriate action was not found.
  
------------------------------------------
01-identification
------------------------------
# Identification

Currently identification only allows user token, Oauth2 bearer tokens are planned for certain endpoints.

To identify a user append to the request an `authorization` header.

The server could respond with the following errors if the validation of the `authorization` header failed:

- **403**: The user that owns the provided token does not have the proper scopes (permissions) to execute this action.

- **401**: The user has failed the header validation

------------------------------------------

02-applications

------------------------------
# Applications

- ## Applications are a preview feature at the moment.

### Get application: applications.getApplication GET /applications/:application_id (200 OK)

- _This endpoint allows you to access public information of an application_

  ```jsonc
  {
    "status": true,
    "application": {
      "id": "string",
      "callback_urls": "string[]",
      "image": "string",
      "name": "string"
    }
  }
  ```
  ------------------------------------------

03-radio

------------------------------

# Radio

- ## Radio is a simple API that provides what currently is playing

### Get currently playing: radio.getCurrent GET /radio/create (200 OK)

- _This endpoint allows you to access public information of what the radio is currently playing_

  ```jsonc

  {

    "status": true,

    "playing": "string | null"

  }

  ```

  ------------------------------------------

04-users

------------------------------

# Users

- ## Users endpoints are the main feature of snail portal, it allows creating new users, updating them and / or deleting them.

### Create a new user: users.createUser POST /users/create (201 Created)

- _This endpoint allows the client to create a new user_

- To perform this action a body with the following schema is required:

  ```jsonc

  {

    "email": "string", //Should be of an email format

    "name": "string",

    "pass": "string", //Should be longer or equal 15 characters in length and smaller than or equal to 300 characters in length,

    "captcha": "string" //Should be a valid hcaptcha response token

  }

  ```

- If the action is successful a response following this schema should be received.

  ```jsonc

  {

    "status": true,

    "token": "string"

  }

  ```

  - This is the first step to creating a new user, the returned token should be used to validate user with the code received by email

### Validate the new created user: users.validateUser POST /users/validate (200 OK)

- To perform this action a body with the following schema is required:

  ```jsonc

  {

    "verificationCode": "string" //Should be the code received by email from the user

  }

  ```

- If the action is successful a response following this schema should be received.

  ```jsonc

  {

    "status": true

  }

  ```

### Create a new token for an already existing user: users.loginUser POST /users/login (201 Created)

- To perform this action a body with the following schema is required:

  ```jsonc

  {

    "email": "string", //Should be a valid user email

    "pass": "string", //Should be the correct user password

    "captcha": "string" //Should be a valid hcaptcha token

  }

  ```

- If the action is successful a response following this schema should be received.

  ```jsonc

  {

    "status": true,

    "token": "string"

  }

  ```

### Identify the current user: users.identifyCurrentUser GET /users/@me (200 OK)

- If the action is successful a response following this schema should be received.

  ```jsonc

  {

    "status": true,

    "user": {

      "id": "string",

      "createdAt": "number",

      "scopes": "string[]",

      "email": "string",

      "name": "string",

      "verified": "boolean"

    }

  }

  ```

### Modify the current user: users.modifyCurrentUser PATCH /users/@me (200 OK)

- To perform this action send a body following this schema

  ```jsonc

  {

    "email"?:"string",//Should be a valid new email that is neither already used or the current user's email.

    "name"?: "string",//Should not be equal to the user's current name

    "pass"?: "string",//Should not be equal to the user's current password,

    "originalPassword": "string",//Should be the user's current password to validate their identity.

    "captcha": "string"//Should be a valid hcaptcha response token.

  }

  ```

  **email,name,pass are all optional properties, but at least one of them should be passed**

  - If the action is successful a response following this schema should be received.

  ```jsonc

  {

    "status": true,

    "verified": "boolean",

    "updated": "(email|name|pass)[]"

  }

  ```

### Begin a deletion sequence for the current user: users.beginDeletionSequence DELETE /users/@me (100 Continue)

- To perform this action the user should not have started a deletion sequence.

- If the action is successful a response following this schema should be received.

  ```jsonc

  {

    "status": true

  }

  ```

### Finish a deletion sequence for the current user: users.finishDeletionSequence POST /@me/delete

- To perform this action the user should have already started a deletion sequence and send a body following this schema

  ```jsonc

  {

    "deleteCode": "string" //Should be the deletion code received by email

  }

  ```

- If the action is successful a response following this schema should be received.

  ```jsonc

  {

    "status": true

  }

  ``` 

  
  