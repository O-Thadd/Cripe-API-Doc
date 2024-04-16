# Cripe API Reference V2
The Cripe API is organized around REST. Our API has predictable resource-oriented URLs, accepts URL parameters, returns JSON-encoded responses, and uses standard HTTP response codes, and verbs.

## Responses
Requests that do not require response data will only return a status and header with an empty response body.
Requests that require response data will always get a json with the following attributes:
- **status:**
  Either 'operation_failure' or 'operation_success' depending on outcome of request
- **response_data:**
  If request succeeded, this will contain the expected data. it will be null otherwise
- **error_message:**
  If request failed, this will contain a helpful message to fix the error. It will be null otherwise

## Pagination
Lists are always paginated. Requests are returned in batches of maximum of 50 items. The concerned endpoints use cursor-based pagination with `lastXid` parameter. Request for the first page does not require the parameter. Subsequent requests should provide the id of the last item in the previous response to get the next batch.
These endpoints will always return a list. Even when a specific item is requested by unique identifier, a list of one item will be returned.
> **Note:**   For requests for posts (both regular posts and posts in rooms):
> - The parameter `lastPostTimestamp` is additionally required.
> - Items are always returned in reverse chronological order. i.e. newest first.

## Push Notification
_...under construction..._

## Enums
- ### post_action

  This is a parameter in the [`POST /posts/post`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#make-a-post) endpoint. Used to specify the action that should be performed on a post.
  - `flame_post_action`: flame a post
  - `unflame_post_action`: unflame a post
  - `vote_post_action`: vote an option in a poll post. The option to vote is provided in the `optionToVote` parameter.
  - `flag_post_action`: flag a post
  - `unflag_post_action`: unflag a post
  - `delete_post_action`: delete a post. Such a request must come from the post owner

- ### room_action

  This is a parameter in the [`POST /rooms/room`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#interact-with-a-room) endpoint. Used to specify the action to perform on a room.
  - `join_room_action`
  - `leave_room_action`
  - `end_room_action`

- ### room_post_action

  This is a parameter in the [`POST /room/posts/post`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#post-message-in-a-room) endpoint. Used to specify the action that should be performed on a post in a room.
  - `flame_room_post_action`: flame a post
  - `unflame_room_post_action`: unflame a post
  - `flag_room_post_action`: flag a post
  - `unflag_room_post_action`: _not a specified feature. not yet impemented..._
  - `delete_room_post_action`: delete a post. Such a request must come from the post owner

- ### user_action

  This is a parameter in the [`POST /users/user`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#interact-with-user) endpoint. Used to specify the action that should be performed on a user.
  - `reset_fcm_token_user_action`: Reset the fcm token. Should be used when a user logs out or deletes account. This stops notifications from being sent to the device.
  - `update_user_action`: This updates a user. Such as for changing username, avatar or password.
  - `delete_user_action`: delete user
- ### inAppMessage type

  This is an attribute in the [InAppMessage](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#inappmessage) object. It indicates the type of the message, if it is a special type; if not, this attribute is null.
  - `new_app_version_app_message`: messages informing users of a new app version will have this value in their `type` attribute
- ### notificationEvent

  This is an attribute in the [NotificationData](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#notificationdata) object. It indicates the event for which the notification is being sent.
  - `mention_notification_event`: when a user is mentioned/tagged in a post.
  - `room_mention_notification_event:` when a user is mention/tagged in a post in a room.
  - `flame_notification_event`: when a user's post is flamed
  - `response_notification_event`: when another user responds/comments/replies to a user's post.

- ### cripe rank

  A user's cripe rank
  - `Anonymous`
  - `Shadow`
  - `Whisperer`
  - `Specter`
  - `Ghost`
  - `Veiled`
  
 
## Objects
### Mention:
This is a nested type in the [post](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#post) object. It represents a user mentioned/tagged in a post.
#### Attributes
- **id:** the userId of the tagged user
- **username:** the username of the tagged user
- **inidices:** _(array of [pair](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#pair)s)_ Each pair represents an exact place in the post that the user is tagged. Indeed, a user can be tagged/mentioned multiple times in the same post. Therefore, the size of the array will depend on the number of times the user is tagged in the post.

### Pair:
This is a nested type in the [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#mention) object. It represents an exact position where a tag/mention of user occurs in a post
#### Attributes
- **first:** _(Number)_ Index of the first letter of an occurrence of the username in the post
- **second:** _(Number)_ second: Index of the last letter of the same occurrence of the username in the post

### InAppMessage
Represents an in-app message
#### Attributes
- **id:** Unique identifier of the message
- **title:** Title of the message
- **body:** Body of the message
- **ctaText:** Text that should appear on the CTA button on the message
- **ctaLink:** URL to visit when the user clicks on the CTA button. Could be null, if message does not have any relevant url
- **type:** Type of message. one of [inAppMessage type](https://github.com/O-Thadd/Cripe-API-Doc/edit/main/API%20Documentation.md#inappmessage-type) enum

### GiftMessage
Represents a gift message
#### Attributes
- **id:** Unique identifier of the message
- **senderId:** Id of user who sent this message
- **body:** Body of the message
- **timestamp:** _(Number)_ Time the message was sent in millis from epoch

### Room
Represents a room
#### Attributes
- **id:** Unique identifier of the room
- **host:** ([User](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#user)) User who started this room i.e. the host
- **title:** Title of the room
- **colour:** Colour theme of the room as hexadecimal RGB triples. e.g. '#FFFFFF' for white, '#800080' for purple, '#000000' for black
- **creationTime:** _(Number)_ Time the room was created in millis from epoch
- **participantCount:** _(Number)_ The number of users in the room
- **peepedUsers:** (Array of [User](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#user)s) Represents 5 or less randomly selected participants in the room

### RoomPost
Represents a message in a room
#### Attributes
- **id:** unique identifier of the post
- **poster:** ([User](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#user)) User who made this post
- **roomId:** id of the room inwhich this post was sent
- **timestamp:** _(Number)_ when post was made in milliseconds from epoch
- **body:** Content of the post
- **flameCount:** _(Number)_ number of flames post has
- **flagCount:** _(Number)_ number of times a post has been flagged. This will always be null except when the request is made by an admin
- **mentions:** (Array of [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#mention)s). Each represents a user tagged/mentioned in the post
- **flamed:** _(boolean)_ indicates if requester flamed this post


### NotificationData
Represents a notification
#### Attributes
- **id:** unique identifier of the notification data
- **actorId:** id of user who performed the action being notified of
- **actorUsername:** username of user who performed the action being notified of
- **event:** the event being notified of. One of [notification event](https://github.com/O-Thadd/Cripe-API-Doc/edit/main/API%20Documentation.md#notificationevent) enum
- **timestamp:** _(Number)_ time of notification in milliseconds from epoch
- **pop:** a suggestion of what to display in the notification for the user
- **postAncestry:** _(array of strings)_ Contains ids of posts in the subject-post's lineage, starting with subject post till the top most post. For a room tag notification, this will contain only two items, viz; first, the id of the post the user was tagged in. second, the id of the room.

  e.g 1. postA has a comment, postB. And postB has a reply, postC. Then a user flames        postC. Notification will be sent to owner of postC about the flaming of the post that    just happened. The post ancestry array will be thus, [postC id, postB id, postA id]
  
  e.g 2. user is tagged in postA inside roomA. The post ancestry will be thus, [postA id, roomA id]
- **data:** contains any additional data. such as content of a gift message

### Post
Represents a post
#### Attributes
- **id:** unique identifier of the post
- **poster:** ([User](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#user)) User who made this post
- **timestamp:** _(Number)_ when post was made in milliseconds from epoch
- **body:** _(Array of Strings)_ Each string represents a page of the post
- **background:** The background colour of the post as hexadecimal RGB triples with a leading '#' e.g. '#FFFFFF' for white, '#800080' for purple, '#000000' for black
- **optionVotePairs:** _(Map)_ Maps an option to number of votes accrued
- **flameCount:** _(Number)_ number of flames post has
- **commentCount:** _(Number)_ number of comments a post has.

  NB: this is only direct comments. Comments of comments are not reflected here
- **flagCount:** _(Number)_ number of times a post has been flagged. this will always be null except when the request is made by an admin
- **duration:** _(Number)_ Applies for a poll. Indicates how long from the time of posting that a poll will remain open to votes.
- **closed:**  _(Boolean)_ Applies for a poll. Indicates if the poll is closed.
- **parentPostId:** Applies if it is a reply/comment. Id of post that this post was made as response to.
- **mentions:** (Array of [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#mention)s). Each represents a user tagged/mentioned in the post
- **flamed:** _(Boolean)_ Indicates if requester flamed this post
- **voted:** _(Boolean)_ Indicates if requester voted this post(poll)

### User
Represents a user
#### Attributes
- **id:** id of user
- **username:** username of user
- **avatarUrl:** Download url for the avatar
- **crips:** _(Number)_ Number of crips the user has. This is earned by activities on the app. And is used to send gift messages to other users.
- **rank:** The cripe rank of this user. One of the [cripe ranks](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#cripe-rank) enum
- **rankProgress:** _(Number)_  A number between 0 and 1, representing the progress of the user in the current rank. This will be null if user is at the highest rank i.e. Veiled


  

## Endpoints
> Parameters are strings except where otherwise specified.


> Endpoints have security. The default is 0 and requires no security data.
> 
> 0:  No security. Requires no extra info.
> 
> 1:  Requires token passed in `x-token` header.
> 
> 2:  Requires credentials. Provided as token as 1 above; or username and password as same-named parameters in the request.

#### Base URl:
- **Production:** _...redacted..._
- **Development:** `https://20240410t125807-dot-thadd-dev-realm.ey.r.appspot.com/v2`



### Posts
#### Make a post
`POST /posts`
- **Description:** makes a new post
- **Security**: 1
- **Parameters:**
  - **body:** _(required)_ (Array of Strings). Each value is a string that is the content of a page in the new post. (when the post is a poll then this array has to have a single value that is the body of the poll, since polls cannot have multiple pages).
  - **options:**  _(Array of Strings)_ Used with a poll. each value is an option for the new poll
  - **duration:** _(Number)_ Used when it is a post. Duration of poll in milliseconds.
  - **parent_postId:** For responses i.e. comments. The id of the post that this post is responding to.
  - **mentions:** _(Array of [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#mention)s)_ Each value is a mention object which represents a user tagged in a post
  - **background:** The background colour of the post as hexadecimal RGB triples. e.g. '#FFFFFF' for white, '#800080' for purple, '#000000' for black

- **returns:** the id of the newly created post
- **Example Response:**
```
{
    "status": "operation_success",
    "error_message": null,
    "response_data": "4e47590d-6bd9-4dc9-a3eb-2dbb435a0761"
}
```

#### Get posts
`GET /posts`
- **Description:** gets posts
- **Security:** 0
  > Token is not required.
  > But to get a tailored response such as wether the user has flagged, flamed or voted in a post; provide a token.
  > The requesting user is inferred from the token, and the response is tailored accordingly. If token is not provided or not valid, those fields will be null.
- **Parameters:**
  - **postId:** id of a post. provide this to get a particular post
  - **posterId:** id of user. provide this to get posts by a particular user.
  - **parent_postId:** id of a post. provide this to get responses/replies to a post.
  - **last_post_timestamp:** _(Number)_ Timestamp of the last post in the previously returned posts. use after the 1st request to get more results. See [pagination](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#pagination)
  - **last_postId:** same use-case as lastTimestamp. the postId of the last post in the previous request.
- **Constraints:**
  - when postId is provided; posterId, parentPostId, lastPostId or lastPostTimestamp should not be provided
  - posterId and parentPostId should not be provided together
- **Returns:** array of requested [Post](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#post) objects
- **Example:**

  success:
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "194e1ec6-81a6-4b5d-80f6-7d48083b265e",
            "posterId": "c27cf047-46d0-48b2-99b7-08a1ee77f4fe",
            "posterUsername": "ulekabo",
            "timestamp": 1712220073854,
            "body": [
                "honestly kai. you sabi"
            ],
            "background": "#000000",
            "optionVotePairs": null,
            "flameCount": 0,
            "commentCount": 1,
            "flagCount": null,
            "duration": null,
            "closed": null,
            "parentPostId": "295de2f6-0bb2-42af-9e15-8b44f1f339ee",
            "mentions": null,
            "flamed": false,
            "voted": null
        },
        {
            "id": "44a2aabd-41b1-4c6f-bf3c-92614bed55b0",
            "posterId": "c27cf047-46d0-48b2-99b7-08a1ee77f4fe",
            "posterUsername": "ulekabo",
            "timestamp": 1712218731266,
            "body": [
                "true talk. no cap"
            ],
            "background": "#000000",
            "optionVotePairs": null,
            "flameCount": 0,
            "commentCount": 0,
            "flagCount": null,
            "duration": null,
            "closed": null,
            "parentPostId": "295de2f6-0bb2-42af-9e15-8b44f1f339ee",
            "mentions": null,
            "flamed": false,
            "voted": null
        },
        {
            "id": "2548a46f-921e-48fe-a130-49741ac2f70a",
            "posterId": "c27cf047-46d0-48b2-99b7-08a1ee77f4fe",
            "posterUsername": "ulekabo",
            "timestamp": 1712186935110,
            "body": [
                "for real mehn. it's so cool"
            ],
            "background": "#000000",
            "optionVotePairs": null,
            "flameCount": 0,
            "commentCount": 0,
            "flagCount": null,
            "duration": null,
            "closed": null,
            "parentPostId": "295de2f6-0bb2-42af-9e15-8b44f1f339ee",
            "mentions": null,
            "flamed": false,
            "voted": null
        },
        ...
    ]
  }
  ```

  Failure:
  ```
  {
      "status": "operation_failure",
      "error_message": "postId should not be provided with any of posterId, parentPostId, lastPostId or lastPostTimestamp",
      "response_data": null
  }
  ```

  #### Interact with a post
  `POST /posts/post`
  - **Description:** performs action on a post
  - **Security:** 1
  - **Parameter:**
    - **postId:** _(Required)_ id of post
    - **post_action:** _(Required)_ action to be taken. one of [post_action](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#post_action) enum.
    - **option_to_vote:** used when voting a post i.e. a poll; with `vote_post_action`. The option to vote for.
  


### Users
#### Register a new user
`POST /users`
- **Description:** creates a new user. For sign up
- **Security:** 0
- **Parameters:**
  - **username:** _(required)_ Username of the new user
  - **password:** _(required)_ Password for the new user
  - **avatar:** _(required)_ Download url for the avatar. Available download urls are gotten from `GET /users/avatars` endpoint.
  - **fcm_token** _(required)_ Firebase Cloud Messagging (FCM) token for the app on the client device. This subserves push-notification feature.
- **Constraints:** Request will fail with appropriate message if a user with same username already exists.
- **Returns:** json with the following fields:
    - userId: unique identifier of the new user
    - token: security token. Required for some requests subsequently
- **Example response:**

Success
```
{
    "status": "operation_success",
    "error_message": null,
    "response_data": {
        "userId": "9ba60ab8-57af-4705-b61d-5371b23e5b30",
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOWJhNjBhYjgtNTdhZi00NzA1LWI2MWQtNTM3MWIyM2U1YjMwIiwiaXNzIjoiY3JpcGVfc2VydmVyIiwiaWF0IjoxNzExNDEwMzkwfQ.PLsGdDugZkvq6tOKd4Hp1Htje5LHqpc8bKnFJqJjnd8"
    }
}
```

  Failure (for non-unique name)
```
{
    "status": "operation_failure",
    "error_message": "username 'mr. ule' is taken. try another username",
    "response_data": null
}
```

#### Get users
`GET /users`
- **Description:** gets users
- **Security:** 0
- **Parameters:**
  - **userId:** userId of user to be retrieved
  - **username:** username of user to be retrieved
  - **search_term:** whole or part of a username. used to search for users by username
  - **last_userId:** the full username of the last user in the previous page. used after the 1st request to get subsequent pages. see [pagination](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#pagination).
- **Constraints:**

  one, and only one of userId, username and searchTerm may be provided
- **Returns:** array of [user](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#user) objects
- **Example response:**

  Success
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "82ec2fb4-a1cc-491f-899f-e5f003ded757",
            "username": "ulekabo",
            "avatar": 19,
            "crips": 2,
            "rank": "Anonymous",
            "rankProgress": 0.02
        },
        {
            "id": "fc973c0a-5494-42ac-8684-1ea6a99839a8",
            "username": "master ule",
            "avatar": 2,
            "crips": 14,
            "rank": "Anonymous",
            "rankProgress": 0.16
        }
    ]
  }
  ```

  Failure: providing username and searchTerm together
  ```
  {
    "status": "operation_failure",
    "error_message": "one, and only one of userId, username and searchTerm should be provided",
    "response_data": null
  }
  ```

#### Interact with user
`POST /users/user`
- **Description:** performs action on a user
- **Security:** 2
- **Parameter:**
  - **user_action:** _(Required)_ Action to be taken on the user. One of the [user_action](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#user_action) enum.

  The following parameters only apply with `update_user_action`
  - **username:** New username
  - **avatar:** New avatar url
  - **old_password:** Current password
  - **new_password:** New password
- **Returns:** When there is a password change, returns the userId and new token in the `userId` and `token` fields respectively within a json object. Otherwise, returns null

#### Send gift message
`POST /users/user/giftMessages`
- **Description:** Sends a gift message to a user
- **Security:** 1
- **Parameters:**
  - **recipientId:** Id of the user the message is being sent to
  - **index:** _(Number)_ This is the index of the message to send in the list returned from the [`GET /users/giftMessages`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#get-available-gift-messages) end point.
- **Returns:** The id of the message

#### Get available gift messages
`GET /users/giftMessages`
- **Description:** Gets a list of all available gift messages that a user can send. Index of a message in the returned list is required in the `POST [/users/user/giftMessages`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#send-gift-message) end point
- **Security:** 0
- **Returns:** An array of strings. Each string is a gift message text.


#### Get user's gift messages
`GET /users/user/giftMessages`
- **Description:** Gets a user's gift messages
- **Security:** 1
- **Parameters:**
  - **last_messageId:** The id of the last message in the previous request. see pagination
  - **last_message_timestamp:** The timestamp of the last message in the previous request. see [pagination](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#pagination)
- **Returns:** An array of [GiftMessage](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#giftmessage) objects
- **Example**
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "367e3011-19fc-48a8-9ff0-47c46878f338",
            "senderId": "c27cf047-46d0-48b2-99b7-08a1ee77f4fe",
            "body": "you are a wonderful person",
            "timestamp": 1712218014157
        },
        ...
    ]
  }
  ```
   
#### Get notifications
`GET /users/user/notifications`
- **Description:** gets the user's new notifications
- **Security:** 1
- **Returns:** array of [NotificationData](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#notificationdata) objects for the requesting user.
- **Example response:**
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "1d0211df-2a22-49c7-a821-6d19d94c9326",
            "actorId": "14ae1b60-caba-475d-bb78-b1bdbd6fdc46",
            "actorUsername": "ulekabo",
            "event": "flame_notification_event",
            "timestamp": 1711470607411,
            "pop": "@ulekabo flamed your post: im so happy to be...",
            "postAncestry": [
                "e62a3488-489c-4982-a50d-e2274a7cc016"
            ],
            "data": null
        },
        ...
    ]
  }
  ```
  
> **Note:**
>
> The requesting user is inferred from the provided token
> 
> Maximum of 50 notifications is provided
> 
> Notifications are wiped after each request.


### Rooms
#### Create a room
`POST /rooms`
- **Description:** creates a room
- **Security:** 1
- **Parameters:**
  - **title:** _(Required)_ Title of the room
  - **colour:** Theme colour of the room as hexadecimal RGB triples. e.g. '#FFFFFF' for white, '#800080' for purple, '#000000' for black
- **Returns:** The id of the newly created rooms

#### Get rooms
`GET /rooms`
- **Description:** Gets rooms
- **Security:** 0
- **Parameters:**
  - **last_roomId:** Id of the room in the previous request. See [pagination](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#pagination)
- **Returns:** Array of [Room](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#room) objects
- **Example:**
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "9055998c-a7ca-4e9e-a6b0-a1ea302d53f4",
            "hostId": "fc973c0a-5494-42ac-8684-1ea6a99839a8",
            "title": "bread or cake",
            "colour": "#000000",
            "creationTime": 1712584773259,
            "participantCount": 1,
            "peepedUsers": [
                {
                    "id": "82ec2fb4-a1cc-491f-899f-e5f003ded757",
                    "username": "ulekabo",
                    "avatar": 19,
                    "crips": 0,
                    "rank": "Anonymous",
                    "rankProgress": 0.0
                },
                ...
            ]
        }
    ]
  }
  ```

#### Interact with a room
`POST /rooms/room`
- **Description:** Interacts with a room. Such as joining, leaving, etc
- **Security:** 1
- **Parameters:**
  - **roomId:** Id of the room
  - **room_action:** Action to be taken on a room. One of the [room actions](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#room_action) enum
 
#### Post message in a room
`POST /rooms/room/posts`
- **Description:** Posts a message in a room.
- **Security:** 1
- **Parameters:**
  - **roomId:** _(Required)_ Id of room
  - **body:** _(Required)_ Body of the message
  - **mentions:** Json array of [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#mention) objects representing users tagged in the message
- **Returns:** Id of the message


  #### Get messages in a room
  `GET /rooms/room/posts`
  - **Description:** Gets messages in room
  - **Security:** 1
  - **Parameters:**
    - **roomId:** Id of the room
    - **last_post_timestamp:** _(Number)_ Timestamp of the last post in the previous request. See pagination.
    - **last_postId:** Id of the last post in the previous request. See [pagination](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#pagination)
  - **Returns:** Array of [RoomPost](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#attributes-5) objects
  - **Example:**
    ```
    {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "bbf71c11-5d06-4879-9722-e932f3282891",
            "poster": {
                "id": "995ef95e-c128-42d5-a81c-57b7239d53a3",
                "username": "sir ule",
                "avatarUrl": "https://storage.googleapis.com/download/storage/v1/b/thadd-dev-realm.appspot.com/o/cripe%2Fimages%2Favatar-set%2F16.png?generation=1713183988438053&alt=media",
                "crips": 18,
                "rank": "Anonymous",
                "rankProgress": 0.09
            },
            "roomId": "8cbe34bb-f1e3-4174-8b9f-1ba21dee024d",
            "timestamp": 1713258591055,
            "body": "its cake and theres no possible counter argument",
            "flameCount": 0,
            "flagCount": 0,
            "mentions": null,
            "flamed": false
        }
        ...
    ]
    }
    ```


 #### Interact with a message in a room
 `POST /rooms/room/posts/post`
 - **Description** Performs action on a post in a room
 - **Security:** 1
 - **Parameters:**
   - **roomId:** Id of the room
   - **postId:** Id of the post/message
   - **room_post_action:** Action to be taken on the post. One of the room post action enums 



### Miscellaneous
#### Log in a user
`GET auth/login`
- **Description:** logs a user in
- **Security:** 0
- **Parameters:**
  - **username:** _(Required)_ username of the user
  - **password:** _(Required)_ password of the user
  - **fcmToken:** _(Required)_ Firebase Cloud Messaging (FCM) token. This subserves push-notification feature
- **Returns:** JSON with the following fields:
  - userId
  - token
- **Example Response**
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": {
        "userId": "9ba60ab8-57af-4705-b61d-5371b23e5b30",
        "token": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ1c2VyX2lkIjoiOWJhNjBhYjgtNTdhZi00NzA1LWI2MWQtNTM3MWIyM2U1YjMwIiwiaXNzIjoiY3JpcGVfc2VydmVyIiwiaWF0IjoxNzExNTczMTg5fQ.TVAgZusskCja6g9YP1jGjVO2ciZqfGAdgGQfFG7zjDk"
    }
  }
  ```

#### Get in-app messages
`GET misc/app/messages`
- **Description:** gets pending in-app messages for a user
- **Security:** 1
- **Returns:** Array of InAppMessages objects
- **Example Response:**
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "72e285b1-13fa-46fc-bde5-360c2093717e",
            "title": "Cripe 2.0 is here",
            "body": "The awesome Cripe V2 is finally available. Update from the Playstore to enjoy wonderful new features",
            "ctaText": "Update",
            "ctaLink": "https://play.google.com/store/apps/details?id=com.vent.cripe",
            "type": "new_app_version_app_message"
        },
        ...
    ]
  }
  ```

#### Get current app version
`GET misc/app/currentVersion`
- **Description:** gets the current version of the mobile app
- **Security:** 0
- **Returns:** latest app verion as string
- **Example Response:**
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": "2"
  }
  ```
