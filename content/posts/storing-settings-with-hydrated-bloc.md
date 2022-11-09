---
title: "Update and persist user settings with hydrated_bloc"
date: 2022-11-08
tags: ["flutter", "dart", "bloc", "hydrated_bloc"]
summary: "How to use hydrated_bloc to update and persist user settings in a Flutter app."
draft: false
showTableOfContents: true
showEdit: true
showAuthor: false
featuredImage: "/posts/hydrated_bloc_thumb.png"
# externalUrl: "https://www.google.com"
---

{{<figure
    src="/posts/hydrated_bloc_thumb.png"
    alt="Hydrated bloc logo"
    caption="[hydrated_bloc](https://pub.dev/packages/hydrated_bloc) by [Felix Angelov](https://github.com/felangel)"
    class="center_scaled">}}


{{<lead>}}
In this post, I'll show you how to use [hydrated_bloc](https://pub.dev/packages/hydrated_bloc) to update and persist user settings like theme, language, etc. in a Flutter app.<br>
{{</lead>}}
This post was made using [hydrated_bloc version 9.0.0](https://pub.dev/packages/hydrated_bloc/versions/9.0.0)

## What is hydrated_bloc? :monocle_face:

hydrated_bloc is a package that extends the [bloc](https://pub.dev/packages/bloc) package to persist state changes to disk. This allows you to persist user settings, such as theme, language, and other preferences.<br>
hydrated_bloc uses [hive](https://pub.dev/packages/hive) as the underlying storage mechanism, which is a fast, NoSQL database that runs on mobile, desktop, and the web.

## Setup hydrated_bloc :hammer_and_wrench:

To use **hydrated_bloc**, you need to add it to your `pubspec.yaml` file:

```yaml
dependencies:
  hydrated_bloc: ^9.0.0
```

Then, you need to initialize the storage. You can use the `HydratedStorage.build()` method to create a storage instance. You can then pass this instance to the `HydratedBloc.storage` property.<br>
In this example, we will make use of the package `path_provider` to get the path to the app's document directory and the `HydratedStorage.webStorageDirectory` for web. We will then use this paths to initialize the storage:

```dart
void main() {
  WidgetsFlutterBinding.ensureInitialized();
  HydratedBloc.storage = await HydratedStorage.build(
    storageDirectory: kIsWeb
        ? HydratedStorage.webStorageDirectory // for web
        : await getApplicationDocumentsDirectory(), // everything else
  );

  // ... runApp
}
```

## Using hydrated_bloc :dart:

Now that we have our storage initialized, we can start using hydrated_bloc. We will start by creating a `Settings` class, that will be used to store the user settings:

```dart
@immutable
class Settings extends Equatable {
  const Settings({
    required this.themeMode,
  });

  final ThemeMode themeMode;
  // ... place other settings here

  Settings copyWith({ThemeMode? themeMode}) =>
      Settings(themeMode: themeMode ?? this.themeMode);

  Map<String, dynamic> toJson() => {'themeMode': themeMode.index};

  factory Settings.fromJson(Map<String, dynamic> map) =>
      Settings(themeMode: ThemeMode.values[map['themeMode'] as int]);

  @override
  bool get stringify => true;

  @override
  List<Object> get props => [themeMode];
}
```

Now that our user settings are defined, we can create a `SettingsCubit` that extends `HydratedCubit`, this cubit will be responsible for updating and persisting the user settings.<br>
We will add a `toggleThemeMode` method to the cubit, that will be used to update the theme:

```dart
class SettingsCubit extends HydratedCubit<Settings> {
  SettingsCubit() : super(const Settings(themeMode: ThemeMode.system));

  void toggleThemeMode(ThemeMode themeMode) =>
      emit(state.copyWith(themeMode: themeMode));

  @override
  Settings fromJson(Map<String, dynamic> json) =>
      Settings.fromJson(json);

  @override
  Map<String, dynamic> toJson(Settings state) => state.toJson();
}
```

{{<alert>}}
**Important**: Persisting multiple instances of the same cubit
{{</alert>}}

hydrated_bloc has a really cool feature which allows us to override a given `id`, this is useful when we want to have multiple instances of the same cubit. For example, if we want to have a `SettingsCubit` for each user, we can override the `id` property to use the user's id:

{{<highlight dart "hl_lines=12">}}
// ...
final carlosCubit = SettingsCubit('carlos');
final dimaCubit = SettingsCubit('dima');
// ...

class SettingsCubit extends HydratedCubit<Settings> {
  SettingsCubit(this._id) : super(const Settings(themeMode: ThemeMode.system));

  final String _id;

  @override
  String get id => _id;
  //... other methods
}
{{</highlight>}}



Since the data we want to persist is the user settings is safe to say that providing this at root level is a good idea.<br>
We can do so by using a `BlocProvider`, thanks to the `flutter_bloc` package _(dont forget to add it to your `pubspec.yaml`)_:

{{<highlight dart "hl_lines=6">}}
void main() {
  // ... Setup storage
  runApp(
    // Provides the settings cubit to the root
    BlocProvider(
      create: (_) => SettingsCubit(),
      child: const MyApp(),
    ),
  );
}
{{</highlight>}}


Having the `SettingsCubit` available at root level, we can now use `context.select` to get the current theme mode and use it to set the theme mode of our app:


{{<highlight dart "hl_lines=11">}}
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hydated Storage Demo',
      theme: ThemeData.light(),
      darkTheme: ThemeData.dark(),
	  // It only listen to the themeMode of the cubit
      themeMode: context.select((SettingsCubit c) => c.state.themeMode),
      home: const SettingsPage(),
    );
  }
}
{{</highlight>}}

Finally, let's create a `SettingsPage` that will allow the user to change the theme mode:

```dart
class SettingsPage extends StatelessWidget {
  const SettingsPage({super.key});

  @override
  Widget build(BuildContext context) {
    final themeMode = context.select(
      (SettingsCubit c) => c.state.themeMode,
    );

    return Scaffold(
      appBar: AppBar(
        title: const Text('Settings Page'),
      ),
      body: Center(
        child: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            Text('Current theme mode: $themeMode'),
			// Using the ThemeMode enum to get the available options
            ...List.generate(
              ThemeMode.values.length,
              (index) {
                final themeMode = ThemeMode.values[index];

                return ElevatedButton(
                  onPressed: () =>
                      context.read<SettingsCubit>()
					  .toggleThemeMode(themeMode),
                  child: Text(themeMode.name),
                );
              },
            ),
          ],
        ),
      ),
    );
  }
}
```

<table>
    <tr>
		<img src="/posts/hydrated_bloc_demo.gif" alt="hydated_bloc demo" width="40%" style="border-radius: 1%; float: left; margin: 0 5% 0 0;">
    </tr>
    <tr>
		<div>
			<h3>And... that's it! :tada:</h3><br>
			Our user settings are now persisted to disk and we could use them to update the app's theme, language, etc.<br><br>
			Easy, right? :smile:
		</div>
    </tr>
</table>

## Conclusion :memo:

In this post, I showed you how to use hydrated_bloc to **update** and **persist** user settings in a Flutter app. I hope you found this post useful. If you have any questions, feel free to leave a comment below.<br>
Thanks for reading! :nerd_face:

## Extra :gift:

The full **source code** with **100% test coverage** :test_tube: for this post is available [here](https://github.com/cgutierr-zgz/stoing_settings_with_hydrated_bloc) :mag:<br>

The **pubspec.yaml** file for this project uses the following dependencies :package:

```yaml
dependencies:
  bloc: ^8.1.0
  equatable: ^2.0.5 # Used to compare objects
  flutter:
    sdk: flutter
  flutter_bloc: ^8.1.1 # Used to provide the cubit to the root
  hydrated_bloc: ^9.0.0 # Used to persist the cubit state
  path_provider: ^2.0.11 # Used to get the storage directory path

dev_dependencies:
  bloc_test: ^9.1.0 # Used to test the cubit
  flutter_test:
    sdk: flutter
  mocktail: ^0.3.0 # Used to mock the storage
  path: ^1.8.2 # Used to create a mock storage
  very_good_analysis: ^3.1.0 # Used to enforce very good practices 🦄
```

## References :books:

- [bloc](https://pub.dev/packages/bloc)
- [equatable](https://pub.dev/packages/equatable)
- [flutter_bloc](https://pub.dev/packages/flutter_bloc)
- [hydrated_bloc](https://pub.dev/packages/hydrated_bloc)
- [hive](https://pub.dev/packages/hive)
- [path_provider](https://pub.dev/packages/path_provider)
- [bloc_test](https://pub.dev/packages/bloc_test)
- [mocktail](https://pub.dev/packages/mocktail)
- [path](https://pub.dev/packages/path)
- [very_good_analysis](https://pub.dev/packages/very_good_analysis)
