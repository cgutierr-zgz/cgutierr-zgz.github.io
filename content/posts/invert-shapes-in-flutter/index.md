---
title: "Using dio and interceptors to refresh tokens and retry failed requests"
date: 2022-11-15
tags: ["flutter", "dart", "dio"]
summary: "Learn how to use dio and interceptors to refresh tokens and retry failed requests in a Flutter app."
draft: true
showTableOfContents: true
showEdit: true
showAuthor: true
# externalUrl: "https://www.google.com"
---
[dio](https://pub.dev/packages/dio) by [Felix Angelov](https://flutterchina.club)

{{<lead>}}
In this post, I'll show you how to use dio to make HTTP requests and how to use interceptors to refresh tokens and retry failed requests, everything while easily logging the requests and responses.
{{</lead>}}
This post was made using:
- [dio version 4.0.6](https://pub.dev/packages/dio/versions/4.0.6)
- [dio_smart_retry version 1.3.2](https://pub.dev/packages/dio_smart_retry/versions/1.3.2)
- [pretty_dio_logger version 1.1.1](https://pub.dev/packages/pretty_dio_logger/versions/1.1.1)

## What is dio? :monocle_face:

Dio is a powerful Http client for Dart, which supports Interceptors, Global configuration, FormData, Request Cancellation, File downloading, Timeout etc.

## What is an interceptor? :monocle_face:

Interceptors are a way to intercept and modify http requests before they are sent to the server and to intercept and modify http responses before they are returned to the caller.

## What is a logger? :monocle_face:

A logger is a way to log the requests and responses to the console.

## What is a retry? :monocle_face:

A retry is a way to retry a failed request.

## What is a token? :monocle_face:

A token is a way to authenticate a user.

## What is a refresh token? :monocle_face:

A refresh token is a way to refresh a token.

## Setting the whole thing :hammer_and_wrench:

To use **dio**, you need to add it to your `pubspec.yaml` file:

```yaml
dependencies:
  dio: ^9.0.0
  dio_smart_retry: ^1.3.2 # optional
  pretty_dio_logger: ^1.1.1 # optional
```

The `dio_smart_try` and `pretty_dio_logger` packages are optional, but I'll be using them in this post, it's an easy way to log the requests and responses and to retry failed requests, but we will make our own retry interceptor for the refresh token part.


////////

GENERAL DIO CLIENT:


class DioClient extends DioForNative {
  DioClient({
    List<Interceptor>? interceptors,
    BaseOptions? options,
  }) : super(
          options ??
              BaseOptions(
                connectTimeout: _defaultTimeoutInMillis,
                sendTimeout: _defaultTimeoutInMillis,
                receiveTimeout: _defaultTimeoutInMillis,
              ),
        ) {
    this.interceptors.clear();
    this.interceptors.addAll(
          interceptors ??
              // Retry Interceptor
              [RetryInterceptor(dio: this, logPrint: print)],
        );

    // Network Logger interceptor only for debug mode
    if (Environment.debug) {
      this.interceptors.add(
            PrettyDioLogger(
              requestHeader: true,
              requestBody: true,
              responseHeader: true,
            ),
          );
    }
    //this.interceptors.add(LoggerInterceptor());
  }

  static const _defaultTimeoutInMillis = 30 * 1000;
}

// The PrettyDioLogger looks way better
// Except for the ResponseType.bytes
/*
class LoggerInterceptor extends LogInterceptor {
  LoggerInterceptor()
      : super(
          requestBody: true,
          responseHeader: false,
          responseBody: true,
          logPrint: (log) {
            if (Environment.debug == false) return;

            debugPrint('🌐 ${log.toString()}');
          },
        );
}
*/


//////////
GENERAL AUTH REPO WITH REFRESH TOKENS ETC...


class AuthRepository {
  AuthRepository({
    required AuthProvider authProvider,
    required FlutterSecureStorage flutterSecureStorage,
    required Dio client,
  }) {
    _authProvider = authProvider;
    _secureStorage = flutterSecureStorage;

    _client = client
      ..interceptors.add(
        InterceptorsWrapper(
          onRequest: ((request, handler) async {
            // We add the accessToken to the headers if it's not null
            final accessToken = await _secureStorage.read(key: _accessTokenKey);

            if (accessToken != null) {
              request.headers['Authorization'] = 'Bearer $accessToken';
            }
            log('[DIO]: Added accessToken [${accessToken != null}]');

            return handler.next(request);
          }),
          onError: (e, handler) async {
            // If the statuscode is 401 we try to refresh the token
            if (e.response?.statusCode == 401) {
              // We refresh the token
              await refreshTokens();

              // We add the accessToken to the headers if it's not null
              final accessToken =
                  await _secureStorage.read(key: _accessTokenKey);

              if (accessToken != null) {
                log('[DIO]: Refreshed Tokens');
                e.requestOptions.headers['Authorization'] =
                    'Bearer $accessToken';

                // Create request with new access token
                final opts = Options(
                  method: e.requestOptions.method,
                  headers: e.requestOptions.headers,
                );
                final cloneReq = await _client.request<void>(
                  e.requestOptions.path,
                  options: opts,
                  data: e.requestOptions.data,
                  queryParameters: e.requestOptions.queryParameters,
                );

                return handler.resolve(cloneReq);
              }
              log("[DIO]: Couldn't refresh Tokens");
            }
          },
        ),
      );
  }

  late final AuthProvider _authProvider;
  late final FlutterSecureStorage _secureStorage;
  late final Dio _client;

  static const _accessTokenKey = 'ACCESS_TOKEN';
  static const _idTokenKey = 'ID_TOKEN';
  static const _refreshTokenKey = 'REFRESH_TOKEN';

  Future<User?> checkSession() async {
    final accessToken = await _secureStorage.read(key: _accessTokenKey);

    if (accessToken == null) return null;

    final user = await _getUserData();

    return user;
  }

  Future<User?> signIn() async {
    final response = await _authProvider.getAuthToken();

    if (response == null) return null;

    await _saveCredentials(
      accessToken: response.accessToken,
      idToken: response.idToken,
      refreshToken: response.refreshToken,
    );

    final user = await _getUserData();

    return user;
  }

  Future<User?> refreshTokens() async {
    final refreshToken = await _secureStorage.read(key: _refreshTokenKey);
    if (refreshToken != null) {
      final response = await _authProvider.refreshTokens(refreshToken);

      if (response == null) return null;

      await _saveCredentials(
        accessToken: response.accessToken,
        idToken: response.idToken,
        refreshToken: response.refreshToken,
      );

      final user = await _getUserData();

      return user;
    }
    await deleteCredentials();
    return null;
  }

  Future<User> _getUserData() async {
    final response = await _authProvider.getUserDetails();

    final json = jsonDecode(jsonEncode(response.data)) as Json;
    final user = User.fromJson(json);

    return user;
  }

  Future<bool> signOut() async {
    //final accessToken = await _secureStorage.read(key: _accessTokenKey);
    await deleteCredentials();

    final response = await _authProvider.signOut(/*accessToken!*/);

    if (response == null) return false;
    return true;
  }

  Future<void> _saveCredentials({
    String? accessToken,
    String? idToken,
    String? refreshToken,
  }) async {
    await _secureStorage.write(key: _accessTokenKey, value: accessToken);
    await _secureStorage.write(key: _idTokenKey, value: idToken);
    await _secureStorage.write(key: _refreshTokenKey, value: refreshToken);
  }

  Future<void> deleteCredentials() async {
    await _secureStorage.delete(key: _accessTokenKey);
    await _secureStorage.delete(key: _idTokenKey);
    await _secureStorage.delete(key: _refreshTokenKey);
  }

  Future<Uint8List?> getPhoto(User user) async {
    final response = await _authProvider.getPhoto(user);

    if (response?.statusCode == 200) return response?.data as Uint8List;
    return null;
  }
}
