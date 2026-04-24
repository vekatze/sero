# sero

`sero` is a [Neut](https://vekatze.github.io/neut/) module that saves values to files and loads them from there.

## Installation

```sh
neut get sero https://github.com/vekatze/sero/raw/main/archive/0-1-46.tar.zst
```

## Types

### Main Definitions

```neut
// Specifies how to encode/decode values.
data sero(a) {
| Sero(
    put: (k: &put-kit, value: &a) -> unit,
    get: (k: &get-kit) -> ?+a,
  )
}

// Encodes a value into a binary value.
define encode<a>(m: sero(a), value: &a, buffer-size: int) -> binary

// Decodes a value from a binary value.
define decode<a>(m: sero(a), bytes: binary) -> ?a
```

### Utility Functions

```neut
// Encodes a value into a binary and saves it to a file at `path`.
define encode-file<a>(m: sero(a), value: &a, buffer-size: int, path: &string) -> system(unit)

// Decodes a value from the file at `path`.
define decode-file<a>(m: sero(a), path: &string) -> system(?a)
```

### Instances

```neut
constant unit-sero: sero(unit)

constant bool-sero: sero(bool)

constant int8-sero: sero(int8)
constant int16-sero: sero(int16)
constant int32-sero: sero(int32)
constant int64-sero: sero(int64)

constant float16-sero: sero(float16)
constant float32-sero: sero(float32)
constant float64-sero: sero(float64)

constant rune-sero: sero(rune)

constant string-sero: sero(string)

constant binary-sero: sero(binary)

inline either-sero<a, b>(!m1: sero(a), !m2: sero(b)) -> sero(either(a, b))

inline pair-sero<a, b>(!m1: sero(a), !m2: sero(b)) -> sero(pair(a, b))

inline list-sero<a>(!m: sero(a)) -> sero(list(a))

inline vector-sero<a>(!m: sero(a)) -> sero(vector(a))
```

## Example

```neut
import {
  core.int.io {print-int-line},
  core.list {for},
  this.decode {decode-file},
  this.encode {encode-file},
  this.instance.int64 {int64-sero},
  this.instance.list {list-sero},
  this.sero {sero},
}

constant _list-int-sero: sero(list(int)) {
  list-sero(int64-sero)
}

define zen() -> unit {
  pin value: list(int) = List[1, 2, 3, 42342, 5, 6];
  let path = "test.bin";
  let encode-result = encode-file(_list-int-sero, value, 10, path);
  match encode-result {
  | Left(e) =>
    // (failed to encode)
    let _ = e;
    Unit
  | Right(_) =>
    let v = decode-file(_list-int-sero, path);
    match v {
    | Left(e) =>
      // (failed to read the file)
      let _ = e;
      Unit
    | Right(v) =>
      match v {
      | Left(_) =>
        // (failed to decode)
        Unit
      | Right(xs) =>
        for(xs, (x) => {
          print("right-right: ");
          print-int-line(x)
        })
      }
    }
  }
}
```
