# dynamic_fonts

The `dynamic_fonts` package for Flutter allows you to (relatively) easily add
dynamically loaded web-hosted fonts to your appーjust like the
[google_fonts](https://pub.dev/packages/google_fonts) package, but for any
arbitrary fonts hosted anywhere.

`dynamic_fonts` is a fork of `google_fonts` with more of the API exposed so that
you can specify your own fonts.

## Why would you use this?

If there is a font that you know you're going to use in your app, just bundle it
normally. Don't bother with this package.

But if your app e.g. offers users multiple choices of fonts, and

1. Some fonts may not be used at all, depending on user preferences
2. Bundling all of the fonts would bloat your app too much

then this package gives you a convenient way to load your fonts dynamically and
keep your initial download slim.

## Getting Started

With the `dynamic_fonts` package, `.ttf` or `.otf` files do not need to be
stored in your assets folder and mapped in the pubspec. Instead, they can be
fetched once via http at runtime, and cached in the app's file system. This is
ideal for development, and can be the preferred behavior for production apps
that are looking to reduce the app bundle size. Still, you may at any time
choose to include the font file in the assets, and the Dynamic Fonts package
will prioritize pre-bundled files over http fetching.  Because of this, the
Dynamic Fonts package allows developers to choose between pre-bundling the fonts
and loading them over http, while using the same API.

For example, say you want to use the
[FiraGO](https://github.com/bBoxType/FiraGO) font (*not* in Google Fonts) in
your Flutter app.

First, add the `dynamic_fonts` package to your [pubspec
dependencies](https://pub.dev/packages/dynamic_fonts#-installing-tab-).

To import `DynamicFonts`:

```dart
import 'package:dynamic_fonts/dynamic_fonts.dart';
```

### Registering a font

To register your font with `DynamicFonts`, you will need to supply the family
name and a map of `DynamicFontsVariant` to `DynamicFontsFile`.

- `DynamicFontsVariant` merely indicates font weight and font style (normal vs
  italic)
- `DynamicFontsFile` stores checksum info and provides the URL for the file

If we want to download FiraGO from the repo hosted on GitHub (*don't do this in
a production app!*), the URL depends on the variant, so we need to do something
like this:

```dart
class FiraGoFile extends DynamicFontsFile {
  FiraGoFile(this.variant, String expectedFileHash, int expectedLength)
      : super(expectedFileHash, expectedLength);

  final DynamicFontsVariant variant;

  String get _dir {
    switch (variant.fontStyle) {
      case FontStyle.normal:
        return 'Roman';
      case FontStyle.italic:
        return 'Italic';
    }
    throw Exception('Unknown style: ${variant.fontStyle}');
  }

  @override
  String get url =>
      'https://raw.githubusercontent.com/bBoxType/FiraGO/9882ba0851f88ab904dc237f250db1d45641f45d/Fonts/FiraGO_TTF_1001/$_dir/FiraGO-${variant.toApiFilenamePart()}.ttf';
}
```

Now we can register the font like so, supplying the SHA256 and file sizes for
security. (Actual SHA256 values not shown!)

```dart
DynamicFonts.register(
  'FiraGO',
  [
    FiraGoFile(
      const DynamicFontsVariant(
        fontWeight: FontWeight.w400,
        fontStyle: FontStyle.normal,
      ),
      '0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef',
      804888,
    ),
    FiraGoFile(
      const DynamicFontsVariant(
        fontWeight: FontWeight.w700,
        fontStyle: FontStyle.normal,
      ),
      '0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef',
      807140,
    ),
    FiraGoFile(
      const DynamicFontsVariant(
        fontWeight: FontWeight.w400,
        fontStyle: FontStyle.italic,
      ),
      '0123456789abcdef0123456789abcdef0123456789abcdef0123456789abcdef',
      813116,
    ),
  ].fold<Map<DynamicFontsVariant, DynamicFontsFile>>(
    {},
    (acc, file) => acc..[file.variant] = file,
  ),
);
```

### Using a font

To use `DynamicFonts` with the default TextStyle:

```dart
Text(
  'This is Dynamic Fonts',
  style: DynamicFonts.getFont('FiraGO'),
),
```

To use `DynamicFonts` with an existing `TextStyle`:

```dart
Text(
  'This is Dynamic Fonts',
  style: DynamicFonts.getFont(
    'FiraGO', textStyle: TextStyle(color: Colors.blue, letterSpacing: .5),
  ),
),
```

or

```dart
Text(
  'This is Dynamic Fonts',
  style: DynamicFonts.getFont('FiraGO', textStyle: Theme.of(context).textTheme.headline4),
),
```

To override the `fontSize`, `fontWeight`, or `fontStyle`:

```dart
Text(
  'This is Dynamic Fonts',
  style: DynamicFonts.getFont(
    'FiraGO',
    textStyle: Theme.of(context).textTheme.headline4,
    fontSize: 48,
    fontWeight: FontWeight.w700,
    fontStyle: FontStyle.italic,
  ),
),
```

You can also use `DynamicFonts.getTextTheme()` to make or modify an entire text
theme to use the "FiraGO" font.

```dart
MaterialApp(
  theme: ThemeData(
    textTheme: DynamicFonts.getTextTheme(
      'FiraGO',
      Theme.of(context).textTheme,
    ),
  ),
);
```

Or, if you want a `TextTheme` where a couple of styles should use a different
font:

```dart
final textTheme = Theme.of(context).textTheme;

MaterialApp(
  theme: ThemeData(
    textTheme: DynamicFonts.getTextTheme('FiraGO', textTheme).copyWith(
      body1: DynamicFonts.getFont('OtherFont', textStyle: textTheme.body1),
    ),
  ),
);
```

### Bundling font files in your application's assets

The `dynamic_fonts` package will automatically use matching font files in your
`pubspec.yaml`'s `assets` (rather than fetching them at runtime via HTTP). Once
you've settled on the fonts you want to use:

1. Download the font files from wherever.  You only need to download the weights
and styles you are using for any given family.  Italic styles will include
`Italic` in the filename. Font weights map to file names as follows:

```dart
{
  FontWeight.w100: 'Thin',
  FontWeight.w200: 'ExtraLight',
  FontWeight.w300: 'Light',
  FontWeight.w400: 'Regular',
  FontWeight.w500: 'Medium',
  FontWeight.w600: 'SemiBold',
  FontWeight.w700: 'Bold',
  FontWeight.w800: 'ExtraBold',
  FontWeight.w900: 'Black',
}
```

2. Move those fonts to a top-level app directory (e.g. `dynamic_fonts`).

3. Ensure that you have listed the folder (e.g. `dynamic_fonts/`) in your
   `pubspec.yaml` under `assets`.

Note: If your fonts' names do not match up with the Google Font naming
conventions, you will either have to register them in the `fonts` section of
`pubspec.yaml`, or rename them.

### Licensing Fonts
Be sure to check the licenses of your fonts. For instance FiraGO comes with an
`OFL.txt` file.

Once you've decided on the fonts you want in your published app, you should add
the appropriate licenses to your flutter app's
[LicenseRegistry](https://api.flutter.dev/flutter/foundation/LicenseRegistry-class.html).

For example:
```dart
void main() {
  LicenseRegistry.addLicense(() async* {
    final license = await rootBundle.loadString('dynamic_fonts/OFL.txt');
    yield LicenseEntryWithLineBreaks(['dynamic_fonts'], license);
  });

  runApp(...);
}
```
