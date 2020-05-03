Style Guide for Flutter / Dart Repos
====================================

Introduction
============

This document contains some high-level philosophy and policy decisions for the Flutter / Dart projects, and a description of specific style issues for some parts of the codebase.

For clarity, this document's language follows [RFC 2119](https://www.ietf.org/rfc/rfc2119.txt)'s definitions.

Table of Contents
=================

1. [Ordering](#ordering)
    1. [Imports](#imports)
    2. [Exports](#exports)
    3. [Class](#class)
    4. [Arguments](#arguments)
2. [Extended Rules](#extended-rules)
    1. [Annotations](#annotations)
    2. [Data Models](#data-models)
    3. [Assertions](#assertions)
    4. [Parameters](#parameters)
    5. [Nullability](#nullability)
    6. [Code Flow](#code-flow)
    7. [Style](#style)

Overview
========

For the most part, the code style can be summarized in as "whatever `dartfmt` does".  This guide only addresses the items that `dartfmt` is not opinionated on.


## Ordering


### Imports

Imports must be grouped and ordered alphabetically within each group.  The groups must be in the following grouping order:

1. `dart:...`
2. `package:...`
3. local file references, ordered by path, then filename


### Exports

Exports must be grouped and ordered alphabetically within each group.  The groups must be in the following grouping order:

1. `package:...`
2. local file references, ordered by path, then filename

*Note:* it should be very rare when exporting a package.


### Class

All classes must follow the ordering of members within the class:

1. Constructor / Factories
2. Variables / Constants
    1. `static`
        1. public
            1. `const`
            2. `final`
            3. variable
        2. private
            1. `const`
            2. `final`
            3. variable
    2. local
        1. public
            1. `const`
            2. `final`
            3. variable
        2. private
            1. `const`
            2. `final`
            3. variable
3. `get` / `set` accessors
    1. Follow same order as "Variables / Constants"
4. Methods
    1. `static`
        1. public
        2. private
    2. local
        1. public -- non-build
            1. lifecycle -- Order by stage; init, update, dispose
            2. non-lifecycle -- Order alphabetically
        2. private -- non-build related
            1. lifecycle -- Order by stage; _init, _dispose
            2. non-lifecycle related -- Order alphabetically
        3. private -- build related
            1. Order alphabetically
        4. public -- build
            1. Last method in the class

**Example**

```dart
class FooState extends State<FooWidget> {
  /* 1: Constructor */
  Foo(...);

  /* 2.1.2.1: static const private variable */
  static const int _maxItems = 25;

  /* 2.2.2.2: final local private variable */
  final List<StreamSubscription> _subscriptions = [];

  /* 2.2.2.3: local private variable */
  bool _initialized = false;
  String _status;
  String _title;

  /* 3.1: get / set */
  bool get initialized => _initialized;

  /* 4.1.1: static public method */
  static String parse(String entry) {...}

  /* 4.2.1.1: local public non-build / lifecycle */
  void initialize() {...}
  void dispose() {...}

  /* 4.2.2.1: local private non-build / lifecycle */
  Future<void> _initialize() async {...}

  /* 4.2.2.2: local private non-build / non-lifecycle */
  String _processStatus() {...}
  String _processTitle() {...}

  /* 4.2.2.3: local private build related */
  Widget _buildStatus() {...}
  Widget _buildTitle() {...}

  /* 4.2.2.4: build */
  Widget build(BuildContext context) {...}
}
```


### Arguments

Arguments should be ordered alphabetically, with the sole exception of `child` / `children` / `build` / `itemBuilder`.  The `child` / `children` / `build` / `itemBuilder` argument must always be the last argument in the list.

**Example**

```dart
Widget _buildMyThing({
  BuildContext context,
  String id,
  bool status,
  String title,
}) {...}

Widget build(BuildContext context) {
  return Container(
    color: Colors.white
    height: 200.0,
    width: 300.0,
    child: Column( /* Child should always be last */
      crossAxisAlignment: CrossAxisAlignment.center,
      mainAxisAlignment: MainAxisAlignment.center,
      children: [ /* Children should always be last */
        Text('Hello'), 
        Text('World'),
        Expanded(
          ListView.builder(
            padding: EdgeInsets.all(16.0),
            itemCount: 2,
            itemBuilder: (BuildContext context, int index) => [ /* itemBuilder should be last */
              Text('Hello'),
              Text('List'),
            ][index],
          )
        ),
      ],
    ),
  );
}

```

Extended Rules
==============


## Annotations

Use annotations to provide extra context for both methods and arguments.

**Example**

```dart
void mySpiffyFunction({
  bool active = false,
  @required String title,
}) {...}

@protected
void onlyBeCalledBySubclass() {...}

@mustCallSuper
void overrideMe() {...}

@override
void overridingOther() {...}

void sign({
  @required String method,
  @visibleForTesting DateTime timestamp,
}) {
  timestamp = timestamp ?? DateTime.now();
}
```

## Data Models

Prefer to make models `@immutable`.  When models can change the data, prefer to follow the `copyWith` pattern instead of making the model mutable.

**Example**

```dart

@immutable Name {
  Name({
    @required this.first,
    @required this.last,
  }) : assert(first?.isNotEmpty == true),
    assert(last?.isNotEmpty == true);

  final String first;
  final String last;

  Name copyWith({
    String first,
    String last,
  }) => Name(
    first: first ?? this.first,
    last: last ?? this.last,
  );
}
```


## Assertions

Use assertions to assist other developers with ensuring your code is being used correctly so errors get caught early and often in development.

Keep assertions simple and on-point.  Don't group unreleated or semi-related checks into a single assertion.  More complex assertions are harder for the developer to determine exactly what was violated.

```dart
void mySpiffyFunction({
  bool active = false,
  @required int max,
  @required int min,
  @required String title,
}) {
  /* Good; each assert is testing one conceptual thing */
  assert(active != null);
  assert(max != null);
  assert(min != null);
  assert(min >= 0);
  assert(max >= min);
  assert(title?.isNotEmpty == true);


  /* Bad; assert is grouping multiple concepts */
  assert(min != null && max != null && min >= 0 && min <= max);
}
```


## Parameters

1. Prefer named parameters for all constructors.
2. Prefer named parameters when for methods with 3 or more parameters.
3. For `build` methods, always accept the `BuildContext` as a required parameter, even in `State` objects
    1. When zero or one additional parameter is needed, use required parameters.
    2. When 2+ additional parameters are needed, use named parameters for the additonal parameters.
4. For methods with a single required parameter, and a single optional parameter, it is acceptable to utilize the optional parameter list combined with the required parameter.

**Example**

```dart
/* 1 */
Foo({
  @required String title,
});

Bar({
  @required String id,
  @required String title,
});

/* 2 */
int add({
  @required int first,
  @required int fourth,
  @required int second,
  @required int third,
}) {
  ...
}

/* 3.2 */
Widget _buildContainer(
  BuildContet context,
  {
    @required double height,
    @required double width,
  }
) {
  ...
}

/* 3.1 */
Widget _buildTitle(
  BuildContext context,
  String title,
) {
  ...
}

/* 4 */
Widget _processTitle(String title) {...}
Widget build(BuildContext context) {...}
```


## Nullability

In Dart, everything can be `null`, including `bool`, `int`, and `double`, which takes some getting used to for developers coming from other typed languages.  This can cause some unexpected bugs if you are not careful.  In addition to following the correct analyzer styles, the following rules will help avoid those bugs.

1. Always compare `bool` values directly to `true` or `false`.
2. For strings or collections, compare against `obj?.isNotEmpty == true` instead of `obj != null && obj.length > 0`.
3. Use `??` or `??=` to for null-check operations, especially assignments.
4. Use `?.` for null-check operators

**Example**

```dart
List<String> names;
bool passed;
Person person;

/* 1 - good */
if (passed == true) {
  // will execute if-and-only-if passed is true
}
if (passed != true) {
  // will execute if passed is false or null
}

/* 1 - bad */
if (passed) {
  // This code will never execute...
} else {
  // ...this won't either.

  // The actual result will be that an exception is thrown because null cannot
  // be evaluated as a bool.
}


/* 2 - good */
if (names?.isNotEmpty == true) {
  // Will execute only if at least one item is in the list
}

/* 2 - bad */
if (names != null && names.length > 1) {
  // Less efficient that the example above, will get flagged by the dart
  // analyzer at build time, and will fail the build.
}


/* 3 - good */
passed = passed ?? true;
names ??= <String>[];

/* 3 - bad */
passed = passed == null ? true : passed;
names = names == null ? [] : names;


/* 4 - good */
if (person?.firstNam?.isNotEmpty == true) {
  // Will execute only if person is populated and so is the first name.
}

/* 4 - bad */
if (person != null && person.firstName != null && person.firstName.length > 0) {
  // Less efficient that the example above, will get flagged by the dart
  // analyzer at build time, and will fail the build.
}
```


## Code Flow

Code is easiest to read when it flows from the top to the bottom of the method.  Given that, methods must only have a single `return`, and it is ideally the final statement of a method.  Occationally, the `return` may happen immediately before the start of a `catch` or `finally` block.

If there's a need to abort the execution of a method due to an error, utilize the `throws` clase.  It's best practice to throw a class that implements `Exception` if the error should have been expected and handled by the developer.  If the error is some system level event that shouldn't have happened, but did, then an `Error` class should be used instead.  Although any object can be part of a `throws`, including `Strings`, it's recommended not to do that and to throw a subclass of `Error` or `Exception` instead.

**Rules**

1. Only ever have a single `return` per method.
2. Only `throw` classes that extend `Exception` or `Error`.
3. Utilize `rethrow` when you want to log an error but are not actually handling it.

**Example**
```dart
/* 1 - good */
Person fromDynamic(dynamic map) {
  Person person;

  if (map != null) {
    person = Person(
      firstName: map['firstName'],
      lastName: map['lastName'],
    );
  }

  return person;
}

/* 1 - bad */
Person fromDynamic(dynamic map) {
  if (map == null) {
    // Now multiple breakpoints would be needed to catch the result before
    // returning.
    return null;
  }

  return Person(
    firstName: map['firstName'],
    lastName: map['lastName'],
  );
}


/* 2 - good */
Future<ApiResult> execute(ApiRequest request) async {
  if (online != true) {
    throw OfflineException();
  }

  // do stuff...
}


/* 2 - bad */
Future<ApiResult> execute(ApiRequest request) async {
  if (online != true) {
    throw 'offlne';
  }

  // do stuff...
}

```


## Style

1. Strongly prefer the use of trailing commas.
    1. *Exception*: If the call takes a single parameter and the line does not wrap.
    2. *Exception*: Closing a stacked call if the indent of the opening line matches the closing.

**Example**

```dart

Future<void> _initialize() async {
  _subscriptions.add(_stream.listen((_) {
    ...
  })); /* Exception 1.2 - the start of _subscription matches the indent of the closing entries */

  _doSomething(
    name: 'foo',
    onFinished: () {
      ...
    }, /* Does not qualify for Exception 1.2 */
  ); /* This should start at the same indent as _doSomething */
}

Widget build(BuildContext context) {
  return Container(
    child: Padding(
      padding: EdgeInsets.only(
        left: 16.0,
        right: 16.0, // Trailing Comma
      ),
      child: Text('Foo' /* Exception 1.1 */), // Trailing Comma
    ), // Trailing Comma
  );
}

```