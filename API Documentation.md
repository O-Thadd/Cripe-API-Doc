# Cripe API Reference V2
The Cripe API is organized around REST. Our API has predictable resource-oriented URLs, accepts JSON-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes, and verbs.

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
Pagination parameter for users is `lastUserId` and that for posts is `lastPostId`
> **Note:**   For requests for posts:
> - The parameter `lastPostTimestamp` is additionally required.
> - Items are always returned in reverse chronological order. i.e. newest first.

## Push Notification

## Enums
- ### post_action

  This is a parameter in the [`/posts/post`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#make-a-post) endpoint. Used to specify the action that should be performed on a post.
  - `flame_post_action`: flame a post
  - `unflame_post_action`: unflame a post
  - `vote_post_action`: vote an option in a poll post. The option to vote is provided in the `optionToVote` parameter.
  - `unvote_post_action`: _not a specified feature. not yet impemented..._
  - `flag_post_action`: flag a post
  - `unflag_post_action`: _not a specified feature. not yet impemented..._
  - `delete_post_action`: delete a post. Such a request must come from the post owner

- ### room_action

  This is a parameter in the POST [`/rooms/room`](https://github.com/O-Thadd/Cripe-API-Doc/blob/main/API%20Documentation.md#interact-with-a-room) endpoint. Used to specify the action to perform on a room.
  - `join_room_action`
  - `leave_room_action`

- ### room_post_action

  This is a parameter in the POST `/room/posts/post` endpoint. Used to specify the action that should be performed on a post in a room.
  - `flame_room_post_action`: flame a post
  - `unflame_room_post_action`: unflame a post
  - `flag_room_post_action`: flag a post
  - `unflag_room_post_action`: _not a specified feature. not yet impemented..._
  - `delete_room_post_action`: delete a post. Such a request must come from the post owner

- ### user_action

  This is a parameter in the `/users/user` endpoint. Used to specify the action that should be performed on a user.
  - `reset_fcm_token_user_action`: Reset the fcm token. Should be used when a user logs out or deletes account. This stops notifications from being sent to the device.
  - `delete_user_action`: delete user
- ### inAppMessage type

  This is an attribute in the InAppMessage object. It indicates the type of the message, if it is a special type; if not, this attribute is null.
  - `new_app_version_app_message`: messages informing users of a new app version will have this value in their `type` attribute
- ### notificationEvent

  This is an attribute in the NotificationData object. It indicates the event for which the notification is being sent.
  - `mention_notification_event`: when a user is mentioned/tagged in a post.
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
This is a nested type in the [post](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#post) object. It represents a user mentioned/tagged in a post.
#### Attributes
- **id:** the userId of the tagged user
- **username:** the username of the tagged user
- **inidices:** _(array of [pair](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#pair)s)_ Each pair represents an exact place in the post that the user is tagged. Indeed, a user can be tagged/mentioned multiple times in the same post. Therefore, the size of the array will depend on the number of times the user is tagged in the post.

### Pair:
This is a nested type in the [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#mention) object. It represents an exact position where a tag/mention of user occurs in a post
#### Attributes
- **first:** _(number)_ Index of the first letter of an occurrence of the username in the post
- **second:** _(number)_ second: Index of the last letter of the same occurrence of the username in the post

### InAppMessage
Represents an in-app message
#### Attributes
- **id:** Unique identifier of the message
- **title:** Title of the message
- **body:** Body of the message
- **ctaText:** Text that should appear on the CTA button on the message
- **ctaLink:** URL to visit when the user clicks on the CTA button. Could be null, if message does not have any relevant url
- **type:** Type of message. one of [inAppMessage type](https://github.com/O-Thadd/Cripe-API-Doc/edit/master/API%20Documentation.md#inappmessage-type) enum

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
- **hostId:** Id of the user who started this room i.e. the host
- **title:** Title of the room
- **colour:** Colour theme of the room as hexadecimal RGB triples. e.g. '#FFFFFF' for white, '#800080' for purple, '#000000' for black
- **creationTime:** _(Number)_ Time the room was created in millis from epoch
- **participantCount:** _(Number)_ The number of users in the room
- **peepedUsers:** Array of users representing 5 or less randomly selected participants in the room

### RoomPost
Represent a message in a room
#### Attributes
- **id:** unique identifier of the post
- **posterId:** id of user who made the post
- **roomId:** id of the room inwhich this post was sent
- **timestamp:** _(number)_ when post was made in milliseconds from epoch
- **body:** Content of the post
- **flameCount:** _(number)_ number of flames post has
- **flagCount:** _(number)_ number of times a post has been flagged. this will always be null except when the request is made by an admin
- **mentions:** array of [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#mention)s. Each represents a user tagged/mentioned in the post
- **flamed:** _(boolean)_ indicates if requester flamed this post


### NotificationData
Represents a notification
#### Attributes
- **id:** unique identifier of the notification data
- **actorId:** id of user who performed the action being notified of
- **actorUsername:** username of user who performed the action being notified of
- **event:** the event being notified of. One of [notification event](https://github.com/O-Thadd/Cripe-API-Doc/edit/master/API%20Documentation.md#notificationevent) enum
- **timestamp:** _(number)_ time of notification in milliseconds from epoch
- **pop:** a suggestion of what to display in the notification for the user
- **postAncestry:** _(array of strings)_ Contains ids of posts in the subject-post's lineage, starting with subject post till the top most post.

  e.g. postA has a comment, postB. And postB has a reply, postC. Then a user flames        postC. Notification will be sent to owner of postC about the flaming of the post that    just happened. The post ancestry array will be thus, [postC id, postB id, postA id]
- **data:** contains any additional data. such as content of a gift message

### Post
Represents a post
#### Attributes
- **id:** unique identifier of the post
- **posterId:** id of user who made the post
- **timestamp:** _(number)_ when post was made in milliseconds from epoch
- **body:** _(array of strings)_ Each string represents a page of the post
- **background:** The background colour of the post as hexadecimal RGB triples. e.g. '#FFFFFF' for white, '#800080' for purple, '#000000' for black
- **optionVotePairs:** _(map)_ Maps an option to number of votes accrued
- **flameCount:** _(number)_ number of flames post has
- **commentCount:** _(number)_ number of comments a post has.

  NB: this is only direct comments. Comments of comments are not reflected here
- **flagCount:** _(number)_ number of times a post has been flagged. this will always be null except when the request is made by an admin
- **duration:** _(number)_ applies for a poll. Indicates how long from the time of posting that a poll will remain open to votes
- **closed:**  _(boolean)_ Applies for a poll. indicates if the poll is closed.
- **parentPostId:** applies if it is a reply/comment. Id of post that this post was made as response to.
- **mentions:** array of [Mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#mention)s. Each represents a user tagged/mentioned in the post
- **flamed:** _(boolean)_ indicates if requester flamed this post
- **voted:** _(boolean)_ indicates if requester voted this post(poll)

### User
Represents a user
#### Attributes
- **id:** id of user
- **username:** username of user
- **avatar:** _(number)_ number indicating the avatar of the user
- **crip:** A crip object. Summary of a the user's cripe rank details.

### Crip
Represents a user's cripe rank details
#### Attributes
- **count:** _(Number)_ Total crips this user has
- **rank:** The cripe rank of this user. One of the cripe ranks enum
- **progress:** _(Number)_  A number between 0 and 1, representing the progress of the user in the current rank. This will be null if user is at highest rank i.e. Veiled

  

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
`https://thadd-dev-realm.ey.r.appspot.com/v2`

### Posts
#### Make a post
`POST /posts`
- **Description:** makes a new post
- **Security**: 1
- **Parameters:**
  - **body:** _(required)_ Array of strings. Each value is a string that is the content of a page in the new post. (when the post is a poll then this array has to have a single value that is the body of the poll, since polls cannot have multiple pages).
  - **options:**  _(array of strings)_ Used with a poll. each value is an option for the new poll
  - **duration:** _(number)_ Used when it is a post. Duration of poll in milliseconds.
  - **parentPostId:** For responses i.e. comments. The id of the post that this post is responding to.
  - **mentions:** _(array of [mention](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#mention)s)_ Each value is a mention object which represents a user tagged in a post
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
- **Parameters:**
  - **postId:** id of a post. provide this to get a particular post
  - **posterId:** id of user. provide this to get posts by a particular user.
  - **parentPostId:** id of a post. provide this to get responses/replies to a post.
  - **lastPostTimestamp:** _(Number)_ Timestamp of the last post in the previously returned posts. use after the 1st request to get more results. See [pagination](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#pagination)
  - **lastPostId:** same use-case as lastTimestamp. the postId of the last post in the previous request.
  - **requesterId:** id of requester. Used to tailor response to requester. Such as indicating posts previously flamed, or polls already voted in
- **Constraints:**
  - when postId is provided; posterId, parentPostId, lastPostId or lastPostTimestamp should not be provided
  - posterId and parentPostId should not be provided together
- **Returns:** array of requested [Post](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#post) objects
- **Example:**

  success:
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "e62a3488-489c-4982-a50d-e2274a7cc016",
            "posterId": "9ba60ab8-57af-4705-b61d-5371b23e5b30",
            "timestamp": 1711413166029,
            "body": [
                "im so happy to be on cripe. it is a great app"
            ],
            "optionVotePairs": null,
            "flameCount": 0,
            "commentCount": 0,
            "flagCount": null,
            "duration": null,
            "closed": null,
            "parentPostId": null,
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
    - **post_action:** _(Required)_ action to be taken. one of [post_action](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#post_action) enum.
    - **optionToVote:** used when voting a post i.e. a poll; with `vote_post_action`. The option to vote for.
  


### Users
#### Register a new user
`POST /users`
- **Description:** creates a new user. For sign up
- **Security:** 0
- **Parameters:**
  - **username:** _(required)_ Username of the new user
  - **password:** _(required)_ Password for the new user
  - **avatar:** _(required)_ _(Number)_ Number that represents the avater. Between 1 and 20, for the 20 currently available avatars
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
  - **searchTerm:** whole or part of a username. used to search for users by username
  - **lastUserId:** the full username of the last user in the previous page. used after the 1st request to get subsequent pages. see [pagination](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#pagination).
- **Constraints:**

  one, and only one of userId, username and searchTerm may be provided
- **Returns:** array of [user](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#user) objects
- **Example response:**

  Success
  ```
  {
    "status": "operation_success",
    "error_message": null,
    "response_data": [
        {
            "id": "4a8862aa-e528-4143-868d-81c352bc7e6c",
            "username": "katolu",
            "avatar": 10,
            "crip": {
                "count": 0,
                "rank": "Anonymous",
                "progress": 0.0
            }
        },
        {
            "id": "61a06576-619a-439a-b68a-336d9cef8d55",
            "username": "mr. ule",
            "avatar": 13,
            "crip": {
                "count": 220,
                "rank": "Shadow",
                "progress": 0.3
            }
        },
        {
            "id": "c27cf047-46d0-48b2-99b7-08a1ee77f4fe",
            "username": "ulekabo",
            "avatar": 5,
            "crip": {
                "count": 2,
                "rank": "Anonymous",
                "progress": 0.02
            }
        },
        ...
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
  - **user_action:** _(Required)_ Action to be taken on the user. One of the [user_action](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#user_action) enum
- **Constraints:**
  - userId provided must match owner of the token or password used to make the request

#### Send gift message
`POST /users/user/giftMessages`
- **Description:** Sends a gift message to a user
- **Security:** 1
- **Parameters:**
  - **recipientId:** Id of the user the message is being sent to
  - **body:** Body of the message being sent
- **Returns:** The id of the message


#### Get gift messages
`GET /users/user/giftMessages`
- **Description:** Gets a user's gift messages
- **Security:** 1
- **Parameters:**
  - **last_messageId:** The id of the last message in the previous request. see pagination
  - **last_message_timestamp:** The timestamp of the last message in the previous request. see pagination
- **Returns:** An array of GiftMessage objects
   
#### Get notifications
`GET /users/user/notifications`
- **Description:** gets the user's new notifications
- **Security:** 1
- **Returns:** array of [NotificationData](https://github.com/O-Thadd/Cripe-API-Doc/blob/master/API%20Documentation.md#notificationdata) objects for the requesting user.
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
  - **last_roomId:** Id of the room in the previous request. See pagination
- **Returns:** Array of Room objects

#### Interact with a room
`POST /rooms/room`
- **Description:** Interacts with a room. Such as joining, leaving, etc
- **Security:** 1
- **Parameters:**
  - **roomId:** Id of the room
  - **room_action:** Action to be taken on a room. One of the room actions enum
 
#### Post message in a room
`POST /rooms/room/posts`
- **Description:** Posts a message in a room.
- **Security:** 1
- **Parameters:**
  - **roomId:** _(Required)_ Id of room
  - **body:** _(Required)_ Body of the message
  - **mentions:** Json array of Mention objects representing users tagged in the message
- **Returns:** Id of the message


  #### Get messages in a room
  `GET /rooms/room/posts`
  - **Description:** Gets messages in room
  - **Security:** 1
  - **Parameters:**
    - **roomId:** Id of the room
    - **last_post_timestamp:** _(Number)_ Timestamp of the last post in the previous request. See pagination.
    - **last_postId:** Id of the last post in the previous request. See pagination
  - **Returns:** Array of RoomPost objects


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
