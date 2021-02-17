## [2.0.0-nullsafety.0] - 2021-02-11

* [google_fonts] Migrated the main library to null safety.
  * One method (`getApplicationSupportDirectory`, from `path_provider`), has not been migrated yet.
  * Most tests are not migrated yet, but all tests are passing.
  * Generator is not migrated yet, but generated code is null-safe.
* [google_fonts] Migrated example to null safety.

## [1.1.2] - 2021-01-26

* [google_fonts] Bump dependency constraints for null safety.

## [1.1.1] - 2020-10-02

* [google_fonts] Use conditional imports to separate out web from destkop + mobile file_io implementations.

## [1.1.0] - 2020-07-29

* Fork from [google_fonts](https://pub.dev/packages/google_fonts) 1.1.0
