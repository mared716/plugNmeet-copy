---
title: Create Poll API | plugNmeet API Reference
description: API endpoint documentation for pushing a poll into a live video conference room from an external application. Learn how to create polls, quizzes, and anonymous votes in an active session.
keywords: [api, create poll, poll, quiz, voting, survey, room api, video api, endpoint]
sidebar_position: 9
sidebar_label: Create Poll
---

# Create Poll

Endpoint: `/room/createPoll`

This API allows your backend server to push a complete poll into an active Plug-N-Meet session in real time. While moderators can create polls from within the client, this endpoint lets an external application inject a poll directly into a running session, making it a powerful building block for automated integrations.

This endpoint is ideal for building integrations such as:
*   Launching a live quiz in a classroom from an external learning platform.
*   Collecting audience votes triggered by events in your application.
*   Scheduling recurring polls or surveys from an external tool.

Once the poll is pushed, it is created in the running state immediately:
1.  Every participant in the live room receives the new-poll notification and can vote right away.
2.  The moderator can then close it, publish the results, or reopen it — just like any poll created from within the client.
3.  If `duration` is set, the poll closes automatically when the time runs out.

For this API call to succeed, the session (`room_id`) must be a currently active room. The request fails if the room has not been created or has already ended.

## Request Parameters

| Field        | Type    | Required | Description                                                                                                                              |
| ------------ | ------- | -------- | ---------------------------------------------------------------------------------------------------------------------------------------- |
| room_id      | string  | Yes      | The unique identifier of the active room into which you want to push the poll.                                                            |
| user_id      | string  | No       | The user ID to attribute the poll to. If omitted, it defaults to `external-api`.                                                          |
| question     | string  | Yes      | The poll question. Must not be empty.                                                                                                     |
| [options](#poll-option) | array | Yes | The poll options. At least two are required and each must have non-empty text. See [Poll Option](#poll-option).                           |
| is_anonymous | boolean | No       | If `true`, voting is anonymous: individual voter choices are never stored per-user. Default: `false`.                                     |
| is_multiple  | boolean | No       | If `true`, participants can select multiple options when voting. Default: `false`.                                                        |
| is_quiz      | boolean | No       | If `true`, the poll runs as a quiz. Correct answers are hidden while the quiz is running and revealed in the results after it is closed. A quiz requires at least one option marked as correct. Default: `false`. |
| duration     | number  | No       | Auto-close duration in seconds. If greater than `0`, the poll closes automatically when the time runs out. Maximum: `3600` (60 minutes). Default: `0` (no time limit). |

### Poll Option

Each entry in the `options` array represents one selectable choice.

| Field      | Type    | Required | Description                                                                    |
| ---------- | ------- | -------- | ------------------------------------------------------------------------------ |
| id         | number  | Yes      | A simple sequential number identifying the option (1, 2, 3, ...).              |
| text       | string  | Yes      | The option text. Must not be empty.                                            |
| is_correct | boolean | No       | If `true`, this option is a correct answer. Only meaningful when `is_quiz` is `true`. |

## Example

### Example 1: A Simple Poll

```json
{
  "room_id": "room01",
  "question": "Which topic should we cover next?",
  "options": [
    {
      "id": 1,
      "text": "Advanced whiteboard features"
    },
    {
      "id": 2,
      "text": "Recording and playback"
    }
  ]
}
```

### Example 2: An Anonymous Multi-Select Poll

```json
{
  "room_id": "room01",
  "user_id": "user-42",
  "question": "Which sessions did you attend today?",
  "options": [
    {
      "id": 1,
      "text": "Morning keynote"
    },
    {
      "id": 2,
      "text": "Product workshop"
    },
    {
      "id": 3,
      "text": "Networking session"
    }
  ],
  "is_anonymous": true,
  "is_multiple": true
}
```

### Example 3: A Timed Quiz

```json
{
  "room_id": "room01",
  "question": "What does the HASH-SIGNATURE header contain?",
  "options": [
    {
      "id": 1,
      "text": "An HMAC-SHA256 signature of the request body",
      "is_correct": true
    },
    {
      "id": 2,
      "text": "The API key"
    }
  ],
  "is_quiz": true,
  "duration": 120
}
```

## Response

| Field       | Type    | Description                              |
| ----------- | ------- | ---------------------------------------- |
| status      | boolean | Indicates if the request was successful. |
| msg         | string  | Response message.                        |
| poll_id     | string  | The unique ID of the newly created poll. |
| status_code | string  | Response [status code](https://github.com/mynaparrot/plugnmeet-protocol/blob/main/proto_files/plugnmeet_common_api.proto#L10). |

## Error Responses

On failure, `status` is `false` and `msg` describes the problem. Validation failures return stable, machine-readable keys that the client UI can translate directly:

| msg key                           | Description                                                              |
| --------------------------------- | ------------------------------------------------------------------------ |
| `polls.errors.question-required`  | The `question` field is missing or empty.                                |
| `polls.errors.min-options`        | Fewer than two options were provided.                                    |
| `polls.errors.option-required`    | An option is missing its text.                                           |
| `polls.errors.quiz-needs-correct` | The poll is marked as a quiz, but no option is marked as correct.        |
| `polls.errors.duration-cap`       | The `duration` exceeds the maximum of 3600 seconds (60 minutes).         |

Other common failures are returned as plain messages: a missing `room_id`, or a room that is not currently active (this endpoint requires the session to be running at the moment the poll is pushed).
