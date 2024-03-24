# Cripe API Reference V2
The Cripe API is organized around REST. Our API has predictable resource-oriented URLs, accepts JSON-encoded request bodies, returns JSON-encoded responses, and uses standard HTTP response codes, and verbs.

## Responses
Requests that do not require response data will only return a status and header with an empty response body.
Requests that require response will always get a json with the following attributes:
- **status:**
  either 'operation_failure' or 'operation_success' depending on outcome of request
- **response_data:**
  this is present if request succeeded and will contain the expected data.
- **error_message:**
  this is present if request fails and will contain a helpful message to fix the error

## Pagination
Lists are always paginated. Requests are returned in batches of maximum of 50 items. The concerned endpoints user cursor-based pagination with `lastXid` parameter. Request for the first page does not require the parameter. Subsequent requests should provide the id of the last item in the previous response to get the next batch.
These endpoints will always return a list. Even when a specific item is requested by unique identifier, a list of one item will be returned.
Pagination parameter for users is `lastUserId` and that for posts is `lastPostId`
> **Note:**   For requests for posts:
> - The parameter `lastPostTimestamp` is additionally required.
> - Items are always returned in reverse chronological order. i.e. newest first.

## Push Notification

## Enums
- ### post_action

  This is a parameter in the `/posts/post` endpoint. Used to specify the action that should be performed on a post.
  - **flame_post_action:** flame a post
  - **unflame_post_action:** unflame a post
  - **vote_post_action:** vote an option in a poll post. The option to vote is provided in the `optionToVote` parameter.
  - **unvote_post_action:** _not a specified feature. not yet impemented..._
  - **flag_post_action:** flag a post
  - **unflag_post_action:** _not a specified feature. not yet impemented..._
  - **delete_post_action:** delete a post. Such a request must come from the post owner
- ### user_action

  This is a parameter in the `/users/user` endpoint. Used to specify the action that should be performed on a user.
  - **reset_fcm_token_user_action:** Reset the fcm token. Should be used when a user logs out or deletes account. This stops notifications from being sent to the device.
  - **delete_user_action:** delete user
- ### inAppMessage type

  This is an attribute in the InAppMessage object. It indicates the type of the message if it is a special type, if not, this attribute is null.
  - **new_app_version_app_message:** messages informing users of a new app version will have this value in their `type` attribute
- ### notificationEvent

  This is an attribute in the NotificationData object. It indicates the event for which the notification is being sent.
  - **mention_notification_event:** when a user is mentioned/tagged in a post.
  - **flame_notification_event:** when a user's post is flamed
  - **response_notification_event:** when another user responds/comments/replies to a user's post.
 
## Objects
### Mention:
This represents a user mentioned/tagged in a post.
#### Attributes
- **id:** the userId of the tagged user
- **username:** the username of the tagged user
- **inidices:** an array of pairs. Each pair represents an exact place in the post that the user is tagged. Indeed, a user can be tagged/mentioned multiple times in the same post. Therefore, the size of the array will depend on the number of times the user is tagged in the post.

### Pair:
This represents an exact position where a tag/mention of user occurs in a post
#### Attributes
- **first:** a number which represents the index of the first letter of an occurrence of the username in the post
- **second:** second: a number which represents the index of the last letter of the same occurrence of the username in the post

### InAppMessage
Represents an in-app message
#### Attributes
- **id:** unique identifier of the message
- **title:** title of the message
- **body:** body of the message
- **ctaText:** text that should appear on the CTA button on the message
- **ctaLink:** url to visit when the user clicks on the CTA button. could be null, if message does not have any relevant urls
- **type:** type of message. one of [inAppMessage type] enum
