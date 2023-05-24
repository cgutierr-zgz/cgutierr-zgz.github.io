title: "Retry, log and refresh auth tokens with HTTP"
date: 2022-11-16
tags: ["flutter", "dart", "http", "interceptors", "retry", "logger"]
summary: "Learn how to use http package in Dart to refresh tokens and retry failed requests in a Flutter app."
draft: false
showTableOfContents: true
showEdit: true
showAuthor: true
sourceCode: "https://github.com/cgutierr-zgz/http-interceptors-loggers"

externalUrl: "https://www.google.com"
http by Dart Team



{{<mermaid>}}
graph TD
A[Christmas] -->|Get money| B(Go shopping)
B --> C{Let me think}
B --> G[/Another/]
C ==>|One| D[Laptop]
C -->|Two| E[iPhone]
C -->|Three| F[Car]
subgraph Section
C
D
E
F
G
end
{{</mermaid>}}
{{<mermaid>}}
graph TD
A[UI] -->|User interaction| B(Event)
B --> C{Bloc}
C -->|Event| D[Repository]
D -->|Response| C

C -->|State| A

{{</mermaid>}}


{{<lead>}}
In this post, I'll show you how to use the http package in Dart to make HTTP requests and how to use interceptors to refresh tokens and retry failed requests, everything while easily logging the requests and responses.
{{</lead>}}
This post was made using:

http version 0.12.0+2
retry version 1.1.0
logging version 0.11.3+2
What is http? :monocle_face:
The http package in Dart is a standard library for making HTTP requests. It supports GET, POST, PUT, PATCH and DELETE requests, and it can be used with or without interceptors, making it easy to modify requests and responses.

Concepts we will be covering in this post:

Interceptors
Interceptors are a way to intercept and modify HTTP requests before they are sent to the server and to intercept and modify HTTP responses before they are returned to the caller.
Loggers
A logger is a way to log the requests and responses to the console.
Retries
A retry is a way to retry a failed request.
Token and Refresh Token
A token is a way to authenticate a user and a refresh token is a way to refresh that token when it expires.
Setting up the http package :wrench:
To use the http package, you need to add it to your pubspec.yaml file:

yaml
Copy code
dependencies:
  http: ^0.12.0+2
  retry: ^1.1.0
  logging: ^0.11.3+2
The retry and logging packages are optional, but I'll be using them in this post, as they provide an easy way to log the requests and responses and to retry failed requests, but we will make our own retry logic for the refresh token part.<br>
I also make use of flutter_appauth and flutter_secure_storage in the example found in the repo.

Making our custom HTTP client :rocket:
To make a request, you need to create a Client instance and use the get method to make a GET request:
