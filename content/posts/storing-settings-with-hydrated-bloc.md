---
title: "Update and persist user settings with hydrated_bloc"
date: 2022-11-04
tags: ["flutter", "dart", "bloc", "hydrated_bloc"]
summary: "How to use hydrated_bloc to update and persist user settings in a Flutter app."
draft: false
showTableOfContents: true
showEdit: true
showAuthor: false
# externalUrl: "https://www.google.com"
---

{{<figure
    src="https://github.com/cgutierr-zgz/stoing_settings_with_hydrated_bloc/blob/main/hydrated_bloc_logo.png?raw=true"
    alt="Hydrated bloc logo"
    caption="[hydrated_bloc](https://pub.dev/packages/hydrated_bloc) by [Felix Angelov](https://github.com/felangel)"
    class="center_scaled">}}


{{<lead>}}
In this post, I'll show you how to use [hydrated_bloc](https://pub.dev/packages/hydrated_bloc) to update and persist user settings like theme, language, etc. in a Flutter app.
{{</lead>}}

## What is hydrated_bloc? :monocle_face:

hydrated_bloc is a package that extends the [bloc](https://pub.dev/packages/bloc) package to persist state changes to disk. This allows you to persist user settings, such as theme, language, and other preferences.<br>
hydrated_bloc uses [hive](https://pub.dev/packages/hive) as the underlying storage mechanism, which is a fast, NoSQL database that runs on mobile, desktop, and the web.

## How to use hydrated_bloc :thinking:

To use **hydrated_bloc**, you need to add it to your `pubspec.yaml` file:

```yaml
dependencies:
  hydrated_bloc:
```

Then, you need to initialize the storage. You can use the `HydratedStorage.build()` method to create a storage instance. You can then pass this instance to the `HydratedBloc.storage` property.
In this example, we will make use of the package `path_provider` to get the path to the app's document directory and the `HydratedStorage.webStorageDirectory` for web. We will then use this paths to initialize the storage:

```dart
WidgetsFlutterBinding.ensureInitialized();
HydratedBloc.storage = await HydratedStorage.build(
  storageDirectory: kIsWeb
      ? HydratedStorage.webStorageDirectory // for web
      : await getApplicationDocumentsDirectory(), // everything else
);
```

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

Now that our user settings are defined, we can create a `SettingsCubit` that extends `HydratedCubit`, this cubit will be responsible for updating and persisting the user settings.

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

Since the data we want to persist is the user settings is safe to say that providing this at root level is a good idea, we can do so by using a `BlocProvider`, thanks to the `flutter_bloc` package _(dont forget to add it to your `pubspec.yaml` file)_:

```dart
runApp(
  // Provides the settings cubit to the root
  BlocProvider(
    create: (context) => SettingsCubit(),
    child: const MyApp(),
  ),
);
```






<table>
    <tr>
		<img src="https://github.com/cgutierr-zgz/stoing_settings_with_hydrated_bloc/blob/main/hydrated_bloc_demo.gif?raw=true" alt="hydated_bloc demo" width="40%" style="border-radius: 1%; float: left; margin: 0 5% 0 0;">
    </tr>
    <tr>
		<div>
			<h3>And... that's it! :tada: :tada: :tada:</h3><br>
			Our user settings are now persisted to disk and we can use them to update the app's theme, language, etc.
		</div>
    </tr>
</table>









## Conclusion :memo:

In this post, I showed you how to use hydrated_bloc to persist user settings in a Flutter app. I hope you found this post useful. If you have any questions, feel free to leave a comment below.<br>


## Extra :gift:

The full **source code** with **100% test coverage** :test_tube: for this post is available [here](https://github.com/cgutierr-zgz/stoing_settings_with_hydrated_bloc) :mag:<br>

## References :books:

- [hydrated_bloc](https://pub.dev/packages/hydrated_bloc)
- [hive](https://pub.dev/packages/hive)
- [path_provider](https://pub.dev/packages/path_provider)
- [flutter_bloc](https://pub.dev/packages/flutter_bloc)
