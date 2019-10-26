---
layout: default
navigation_weight: 5
permalink: /project3/
title: Project 3
---

# Project 3: Chat Server and Corresponding React Front-End

In this project you will write a chat server using sinatra and packaged in a
docker container, as well as a standalone React-based front-end for your chat
sever.

Your front-end should have the capability to seamlessly work with both your
server, and my deployed version of the server (reference server). Similarly my
deployed front-end (reference client) should be able to seamlessly interact
with your server.

## Working in Pairs (optional)

This project is significantly more complex than the previous projects. You will
need to maintain state in your Sinatra application, and many of you will learn
an entirely new framework, React. I estimate this project to be more than four
times the work of [Project 2](/project2/). As a result, this project spans two
weeks, and you have the option of working with another student of the course
who is in your same lab section.

If you are to work in a pair, please have one member of your pair send a
private message on Piazza to `instructors` and include your pair in the `to`
indicating that the two of you will work together. Once you've formed a pair,
you and your partner are committing to stick with the pairing.

If you intend to work solo, please also message `instructors` on Piazza to
indicate your choice to work solo. You may later pair up with someone by the
pairing deadline.

The pairing deadline is: Tuesday October 22, 10:59:59 AM PDT.

## Learning Outcomes

- Student has written and deployed a React application coded using JSX.

- Student has added CORS headers to their web application to support
  third-party front-ends.

- Student has leveraged Server-Sent Events and Javascript's EventSource to
  provide a real-time communication platform.

## Due Date

Tuesday October 29, 10:59:59 AM PDT

## Overview Video

[https://youtu.be/FutR00lpAfE](https://youtu.be/FutR00lpAfE)

## Project Submission

Please have each team member submit the following form:
[https://forms.gle/3jDCN4MsaES169DG7](https://forms.gle/3jDCN4MsaES169DG7)

### Deliverables

In order to submit the form you will need three additional things:

- The name of a pushed docker container that when invoked runs your chat server.

- A url to your deployed front-end application with minimized javascript (tip: `yarn build`).

- A url to a repository containing your front-end source code. Assuming this repository is
  `private`, please invite me, `bboe` on github so that I can see your code.

## HTTP API Specification

The following endpoints are the only required endpoints. Feel free to add more
endpoints or more functionality so long as the reference client continues to
work with your server. Sections indicated via `(reference)` are not required,
but are listed so you understand why the reference server behaves a certain
way.

**Note**: I've intentionally excluded any CORS related headers and endpoints
from the following list.

### POST /login

This endpoint is used to grant a user an access token. In the reference
implementation, the endpoint is also used to immediately register a new
user. Once a user has been created, they must login with the same password.

Returns:

- `201` with the JSON body `{"token": <SIGNED TOKEN>}` on success

- `403` if the provided `username` and `password` combination doesn't match
  that of an existing user

- `422` if either `password` or `username` is blank

- (reference) `422` if the set of provided fields do not exactly match the
  expected fields

Expected Form Fields:

- `password`
- `username`

**WARNING**: Do not send real passwords to this system. The reference server is
not protected by TLS and thus all data sent to or from the server is
unencrypted.

Example curl command:

```sh
curl -D- <BASE_URL>/login -F username=<USERNAME> -F password=<PASSWORD>
```

Example HTTP response:

```
HTTP/1.1 201 CREATED
Content-Type: application/json

{"token": "<SIGNED TOKEN>"}
```

### POST /message

Send a message to all users of the chat system.

Returns:

- `201` on success

- `403` if `<SIGNED TOKEN>` is not valid

- `422` if `message` is blank

- (reference) `422` if the set of provided fields do not exactly match the
  expected fields

Expected Headers:

- `Authorization` with value `Bearer <SIGNED TOKEN>`

Expected Form Fields:

- `message`: a string of the message to send

Example curl command:

```sh
curl -D- <BASE_URL>/message -F message=test -H "Authorization: Bearer <SIGNED TOKEN>"
```

Example HTTP response:

```
HTTP/1.1 201 CREATED
```

### GET /stream/&lt;SIGNED TOKEN&gt;

Returns:

- `200` and begins the Server-Sent Event stream with events as described the
  following section

- `403` if `<SIGNED TOKEN>` is not valid

Example curl command:

```sh
curl -D- <BASE_URL>/stream/<SIGNED TOKEN>
```

Example (partial) HTTP response:

```
HTTP/1.1 200 OK
Content-Type: text/event-stream; charset=utf-8

data: {"users": ["curl"], "created": 1570999219.797813}
event: Users
id: 0718299d-43ef-4b1c-b1cf-ba828d195959

data: {"status": "Server start", "created": 1570947584.0895946}
event: ServerStatus
id: ed4e3e63-9680-436d-9ab2-e0546b5cc03f

data: {"message": "We're online!", "user": "bboe", "created": 1570947655.5598643}
event: Message
id: 1a72e044-92e4-4ee5-94fe-5cabacb75b83
```

## SSE Events

Below are a list of events that you must support and implement. The `data`
field of all events must be JSON. All events have a unique ID which is included
as part of the SSE protocal, and not part of the `data` attribute.

### Disconnect

Indicates that the server is closing the connection. The browser must not
auto-retry on disconnect.

Fields:

- `created` (float): the unix timestamp when the event was created

### Join

Indicates that a user has joined the chat.

Fields:

- `created` (float): the unix timestamp when the event was created
- `user` (string): the username of the user who joined the chat

### Message

Represents a message from a user connected to the chat.

Fields:

- `created` (float): the unix timestamp when the event was created
- `message` (string): the message from the user
- `user` (string): the username of the sender

### Part

Indicates that a user has left the chat.

Fields:

- `created` (float): the unix timestamp when the event was created
- `user` (string): the username of the user who left the chat

### ServerStatus

Used for the server to provide status updates.

Fields:

- `created` (float): the unix timestamp when the event was created
- `status` (string): the message from the server

### Users

Provides a complete list of users connected to the chat server. This message is
always sent out on connection of new streams.

Fields:

- `created` (float): the unix timestamp when the event was created
- `users` (array[string]): the list of connected users

## Server Requirements

- Your server must maintain state about the users who are connected.

- A broadcast `JOIN` should be made any time a new user is connected.

- A broadcast `PART` should be made any time a user has disconnected.

- A `Users` event should be sent to each newly established connection.

- Incoming messages should be broadcast to everyone.

- A `Disconnect` should be sent to the existing connection when a user attempts
  to connect with a second client (the second client should remain connected).

  - No `JOIN` or `PART` message should occur in such a case because the user is
    still connected.

- The first event in the server should be `ServerStatus` indicating the server
  has started.

- A history of at least the last 100 events should be kept.

- All of the `Message` or `ServerStatus` in the history should be sent to a
  newly connecting user.

- A user who is reestablishing its connection (retry after failure) should
  receive all of the messages in the history that have occurred since the
  provided `last_event_id`.

## React Front-end Specification

Your application need not be anything like the reference application. It
however, must meet the following requirements:

- Connection status should be visually discernable between being connected and disconnected.

- There should be an easy way to see who is connected.

- There should be a way to discover when someone connected (`JOIN`).

- It should be easy to discover when someone disconencted (`PART`).

- New `Message` events should be immediately apparent.

- `ServerStatus` events should be locatable.

- For retryable connection failures, your application should automatically
  reconnect (`EventSource` should handle this for you).

- On `Disconnect` your application should not automatically attempt to
  reconnect to the server.

- A user should always be able to take an action (e.g., connect, send
  message). In other words there should be no terminal state that requires a
  page refresh.

- Separate browser windows and/or tabs should each be able to have their own
  connection to the server.

### Components

While the names do not need to be the same, you need to at least implement the
following React components:

- Compose (a way to input / send a message)
- LoginForm
- MessageList
- UserList

## Developing React Using Docker

The following instructions are not necessary, but might make it easier if you
don't want to set up the dependencies on your machine.

### Create or change into a directory where you want your project to live under.

```ssh
mkdir project3
cd project3
```

### Create React App

Run the following to start up a node-based container and drop into a shell.

```sh
docker run -it --rm -p 3000:3000 -v $(pwd):/app -w /app node /bin/bash
```

The above maps local port `3000` to container port `3000`. Synchronizes the
contents of the current local directory with `/app` in the container, and
starts up `bash`.

Once running, create your React application inside the container via:

```
npx create-react-app chat_client
```

When that's done, note that in your local directory (not in the container), a
subdirectory named `chat_client` has appeared. This is your React application.

### Start the development server

Connect to the container (re-run the above `docker run` command, if necessary,
and then run:

```sh
cd chat_client
yarn start
```

Once started, you should be able to access your application via:
[http://localhost:3000](http://localhost:3000)

### Make Changes

Locally, edit the contents of files under `chat_client` and when you save, you
should see said changes automatically take effect in the browser without
needing to refresh.

### React Tutorial

Follow this guide to add more components:
[https://reactjs.org/docs/hello-world.html](https://reactjs.org/docs/hello-world.html)

## Resources

### Hosted Server Example

[http://chat.cs291.com/](http://chat.cs291.com/)

The application at the above URL contains a complete server implementation
which your client should be able to communicate with. Of course, the server
side code will not be provided as it's up to you to replicate its
functionality.

### Client Example

While the above link also serves a complete client, it's more interesting to
have a client hosted on a different domain as the interaction then requires
CORS. A copy of the client, with CSS and JavaScript separated can be found at:

[https://cs291.com/project3/chat/](https://cs291.com/project3/chat/)

And, while you can view the source in the browser, it might be more convenient
to see it on GitHub:

[https://github.com/scalableinternetservices/ucsb_website/tree/master/project3/chat](https://github.com/scalableinternetservices/ucsb_website/tree/master/project3/chat)

**Note**: The logic of this client is written 100% in JavaScript and as such it
serves as a poor example of code to copy since you can better accomplish the
same with React. While you may end up writing more code when using React, the
maintainability of the React code is significantly greater, especially when
accompanied with component unit tests.

## Required Tools

- [Cross-Origin Resource Sharing](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS) (CORS)

- [Docker](https://www.docker.com/products/docker-desktop)

- [JSX](https://reactjs.org/docs/introducing-jsx.html)

- JavaScript [EventSource](https://developer.mozilla.org/en-US/docs/Web/API/EventSource)

- [React](https://reactjs.org)

- [Server-Sent Events](https://www.w3.org/TR/eventsource/) (SSE)

- [Sinatra](http://sinatrarb.com/)

## Suggested Reading

- [Server-Sent Events (SSE)](https://hpbn.co/server-sent-events-sse/) in High Performance Browser Networking

- [XMLHttpRequest](https://hpbn.co/xmlhttprequest/) in High Performance Browser Networking
