# Dart Style Guide

## Table of Contents

- [Dart Style Guide](#dart-style-guide)
  - [Table of Contents](#table-of-contents)
  - [Introduction](#introduction)
  - [Guidelines](#guidelines)
    - [Keep your build functions small](#keep-your-build-functions-small)
    - [Use trailing commas](#use-trailing-commas)
  - [Style](#style)
    - [Be Consistent](#be-consistent)
    - [Package and File Names](#package-and-file-names)
    - [Import Aliasing](#import-aliasing)
    - [Comments](#comments)
    - [Avoid Empty Else](#avoid-empty-else)
    - [Declare Return Types](#declare-return-types)

## Introduction

Styles are the conventions that govern our code. The term style is a bit of a misnomer, since these conventions cover far more than just source file formatting pedantic handles that for us.

The goal of this guide is to manage this complexity by describing in detail the
Dos and Don'ts of writing Dart code at Bounce. These rules exist to keep the codebase manageable while still allowing engineers to use Dart language features productively.

This guide is created by [Nandan Satheesh]. 

  [Nandan Satheesh]: https://github.com/NandanSatheesh

This documents idiomatic conventions in Dart code that we follow at Bounce. A lot of these are general guidelines for Dart, while others extend upon external
resources:

1. [Effective Dart](https://dart.dev/guides/language/effective-dart)

## Guidelines

### Keep your build functions small

Seperate out widgets as simple functions where ever possible to keep the build function look small in size. 

```
Widget build(BuildContext context) => Column(  
  children: <Widget>[  
        Row( ... ),
        Container(... ),
        Text(....),
    ]
);        
```

```
Widget getMainRow() => Row(...);

final customContainer = Container(...);

final titleText = Text(...);

Widget build(BuildContext context) => Column(  
  children: <Widget>[  
        getMainRow(),
        customContainer,
        titleText,
    ]
);   
```


### Use trailing commas

For a neat formatting, it's always recommened to add *trailing commas*. Add a trailing comma at the end of a parameter list in functions, methods, and constructors. 

```
Widget build(BuildContext context) => Column(  
  children: <Widget>[Icon(Icons.add))]);        
```

```
Widget build(BuildContext context) => Column(  
  children: <Widget>[  
        Icon(Icons.add),
    ]
);   
```

## Style

### Be Consistent 

Few guidelines written in the document may be situational and subjective. 

Above all else, **be consistent**.

Consistent code is easier to maintain, is easier to rationalize, requires less
cognitive overhead, and is easier to migrate or update as new conventions emerge or classes of bugs are fixed.

Conversely, having multiple disparate or conflicting styles within a single
codebase causes maintenance overhead, uncertainty, and cognitive dissonance,
all of which can directly contribute to lower velocity, painful code reviews,
and bugs.

When applying these guidelines to a codebase, it is recommended that changes
are made at a package (or larger) level: application at a sub-package level
violates the above concern by introducing multiple styles into the same code.

### Package and File Names

When naming packages or a dart file, choose a name that is:

- All lower-case. No capitals. Use underscores for spaces.
- Short and succinct. Remember that the name is identified in full at every call
  site.
- Not plural. For example, `viewmodel`, not `viewmodels`.


### Import Aliasing

Import aliasing should be avoided unless there is a direct conflict between imports.

```
import 'package:http/http.dart' as http;
```

### Comments

Use `///` for documentation only, preferably for members and types. Keep it small and simple. 

```
// We are getting a name.
String get name => ... ;
```

```
///  Gets [name] string.
String get name => ... ;
```

Use `//` for inline comments only if it's necessary. 

```
void print(String name) {
	/* Prints the name. */
	print(name);
}
```

```
void print(String name) {
	// Prints the name.
	print(name);
}
```

### Avoid Empty Else

**AVOID** empty else statements.

```
if (x > y)
  print("1");
else ;
  print("2");
```

```
if (x > y)
  print("1");
print("2");
```

### Declare Return Types

**ALWAYS** declare method return types. 

```
_myMethod() => 42;
```

```
int _myMethod() => 42;
```
