---
title: "Using enhanced enums with go_router"
date: 2022-11-09
tags: ["flutter", "dart", "enums", "go_router"]
summary: "How to use enhanced enums with go_router to create a more type-safe navigation system."
draft: false
showTableOfContents: true
showEdit: true
showAuthor: false
# externalUrl: "https://www.google.com"
---

[go_router](https://pub.dev/packages/go_router) by [Flutter.dev](https://pub.dev/publishers/flutter.dev/packages)

{{<lead>}}
In this post I want to show **how I like** to use the new **enhanced enums** with **go_router**. This can also apply if you use **any other navigator**, I just enjoy using go_router.
{{</lead>}}
This post was made using [go_router version 5.1.5](https://pub.dev/packages/go_router/versions/5.1.5)

## What are enhanced enums? :thinking:

Enhanced enums are a great way to define a set of values that are known at **compile time**. They are similar to the [enum](https://dart.dev/guides/language/language-tour#enumerated-types) type, but they can also have **methods** and **properties**.

_The new [enhanced enums](https://dart.dev/guides/language/language-tour#declaring-enhanced-enums) are available since Dart [2.17](https://dart.dev/guides/language/evolution#dart-217)_

## How to use enhanced enums :dart:

First, ensure your dart/flutter project is using at least Dart 2.17.0 in your `pubspec.yaml`:

```yml
environment:
  sdk: ">=2.17.0 <3.0.0"
```

Let's start with a simple example. We want to define a set of **routes** for our app. We can do this by creating an **enum**:

```dart
enum AppRoutes {
  login,
  home,
  settings,
}
```

Now that we have our enum, we can **enhance** it to define our routes, views, etc...

```dart
enum AppRoutes {
  login('login', LoginPage()),
  home('/home', HomePage()),
  settings('/settings', SettingsPage()); // Don't forget to add the semicolon here :)

  const AppRoutes(
	this.path,
	this.view,
  );

  final String path;
  final Widget view;
}
```

That's it! Now our enum has a **path** and a **view**.

## Combining enhanced enums with go_router :rocket:

Now that we have our enhanced enum, we can use it with [go_router](https://pub.dev/packages/go_router) to define our routes.

Let's first add the `go_router` dependency to our `pubspec.yaml`:

```yml
dependencies:
  go_router: ^5.1.5
```

And now let's create our router:

```dart
final router = GoRouter(
  initialLocation: AppRoutes.login.path,
  routes: [
    GoRoute(
      path: AppRoutes.login.path,
      pageBuilder: (context, state) => MaterialPage<void>(
        key: state.pageKey,
        child: AppRoutes.login.view,
      ),
    ),
	// ... same for home/settings using AppRoutes.x.path and x.home.view
  ],
);
```

This already looks much better than the previous version, where we had to define the routes and views manually:

```dart
final router = GoRouter(
  initialLocation: '/login',
  routes: [
	GoRoute(
	  path: '/login',
	  pageBuilder: (context, state) => MaterialPage<void>(
		key: state.pageKey,
		child: LoginPage(),
	  ),
	),
	// ...
);
```

Anyway, we can make this even better by adding a **method** to our enum so that we can get the route directly from the enum just by calling `AppRoutes.routeX.route`:

```dart
enum AppRoutes {
  // ...
  GoRoute get route => GoRoute(
        path: path,
        pageBuilder: (context, state) => MaterialPage<void>(
          key: state.pageKey,
          child: view,
        ),
      );
}
```

This will allow us to simplify our router:

```dart
final router = GoRouter(
  initialLocation: AppRoutes.login.path,
  routes: [
	AppRoutes.login.route,
	AppRoutes.home.route,
	AppRoutes.settings.route,
  ],
);
```

I like it picasso! :art:

## Adding our router :construction:

Now that we have our router, we can add it to our app by updating our MaterialApp widget with a `.router()` constructor and adding a `routerConfig` parameter:

{{<highlight dart "hl_lines=6 8">}}
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      title: 'Enums and Router demo',
      routerConfig: router,
    );
  }
}
{{</highlight>}}

Now that we have our router, we can easily navigate to those known routes, let's test it by adding a text button to the login page:

```dart
//...
TextButton(
  onPressed: () => context.go(AppRoutes.home.path),
  child: Text('Home'),
),
//...
```

We can even simplify this by adding another method to our enum:

```dart
enum AppRoutes {
  // ...
  void go(BuildContext context) => context.go(path);
  // Additionall, we can add push and replace methods :)
  void push(BuildContext context) => context.push(path);
  void replace(BuildContext context) => context.replace(path);
}
```

And now we can easily navigate to our with the use of our new methods:

```dart
//...
TextButton(
  onPressed: () => AppRoutes.home.go(context),
  // onPressed: () => AppRoutes.home.push(context),
  // onPressed: () => AppRoutes.home.replace(context),
  child: Text('Home'),
),
//...
```

Now navigating seems much easier and less error-prone :smile:

## Conclusion :memo:

In this post, I showed you how I like to use enhanced enums with go_router in order navigate in a more type-safe, less error-prone and _**in my opinion**_ more readable way.<br>
For sure there are some use cases where this approach might not be the best, but I think it's worth trying it out and see if it fits your needs :wink:

I hope you enjoyed it and that you found it useful.<br>
If you have any questions or suggestions, feel free to **leave a comment** below. :smile:<br>
Thanks for reading! :nerd_face:

## Extra :gift:

The full **source code** for this post is available [here](https://github.com/cgutierr-zgz/enhanced_enums_and_go_router) :mag:<br>

The **pubspec.yaml** file for this project uses the following dependencies :package:

```yaml
dependencies:
  flutter:
    sdk: flutter
  go_router: ^5.1.5 # Used to define our router

dev_dependencies:
  flutter_test:
    sdk: flutter
  very_good_analysis: ^3.1.0 # Used to enforce very good practices 🦄
```

## References :books:

- [go_router](https://pub.dev/packages/go_router)
- [Dart enums](https://dart.dev/guides/language/language-tour#enumerated-types)
- [Dart 2.17.0](https://dart.dev/guides/language/evolution#dart-217)
