---
title: "Storing settings with hydrated_bloc"
date: 2022-11-04
tags: ["flutter", "dart", "bloc", "hydrated_bloc"]
summary: "How to use hydrated_bloc to persist user settings in a Flutter app."
draft: false
showTableOfContents: true
showEdit: true
showAuthor: false
# externalUrl: "https://www.google.com"
---

{{<figure
    src="posts/hydrated_bloc_logo.png"
    alt="Hydrated bloc logo"
    caption="[hydrated_bloc](https://pub.dev/packages/hydrated_bloc) by [Felix Angelov](https://github.com/felangel)"
    class="center_scaled">}}

In this post, I'll show you how to use [hydrated_bloc](https://pub.dev/packages/hydrated_bloc) to persist user settings in a Flutter app.


# This is still a {{<skills>}}draft{{</skills>}}
<kbd>TODO:</kbd>

1. Explain how to setup the storage for web too
2. Add a decent settings object to the example


## What is hydrated_bloc?

hydrated_bloc is a package that extends the [bloc](https://pub.dev/packages/bloc) package to persist state changes to disk. This allows you to persist user settings, such as theme, language, and other preferences.<br>
hydrated_bloc uses [hive](https://pub.dev/packages/hive) as the underlying storage mechanism, which is a fast, NoSQL database that runs on mobile, desktop, and the web.

## How to use hydrated_bloc

To use hydrated_bloc, you need to add it to your `pubspec.yaml` file:

```yaml
dependencies:
  hydrated_bloc:
```

Then, you need to initialize the storage. You can use the `HydratedStorage.build()` method to create a storage instance. You can then pass this instance to the `HydratedBloc.storage` property:

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  HydratedBloc.storage = await HydratedStorage.build();
  runApp(MyApp());
}
```

You can then use the `HydratedBloc` class to create your blocs. This class extends the `Bloc` class, so you can use it in the same way as the `Bloc` class.

```dart
class ThemeBloc extends HydratedBloc<ThemeEvent, ThemeState> {
  ThemeBloc() : super(ThemeState.light());

  @override
  Stream<ThemeState> mapEventToState(ThemeEvent event) async* {
	if (event is ThemeChanged) {
	  yield event.theme;
	}
  }

  @override
  ThemeState fromJson(Map<String, dynamic> json) {
	try {
	  return ThemeState.values[json['theme'] as int];
	} catch (_) {
	  return null;
	}
  }

  @override
  Map<String, int> toJson(ThemeState state) {
	return {'theme': state.index};
  }
}
```

## Conclusion

In this post, I showed you how to use hydrated_bloc to persist user settings in a Flutter app. I hope you found this post useful. If you have any questions, feel free to leave a comment below.

## References

- [hydrated_bloc](https://pub.dev/packages/hydrated_bloc)
- [hive](https://pub.dev/packages/hive)
