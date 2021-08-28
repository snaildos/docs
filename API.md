---
title: Official API for SnailDOS
description: Full API regarding SnailDOS
published: true
date: 2021-08-28T14:45:09.110Z
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
01-responses
------------------------------
# Identification

Currently identification only allows user token, Oauth2 bearer tokens are planned for certain endpoints.

To identify a user append to the request an `authorization` header.

The server could respond with the following errors if the validation of the `authorization` header failed:

- **403**: The user that owns the provided token does not have the proper scopes (permissions) to execute this action.

- **401**: The user has failed the header validation