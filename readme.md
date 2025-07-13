# sero

`sero` is a [Neut](https://vekatze.github.io/neut/) module that saves values to files and loads them from there.

## Installation

```sh
neut get sero https://github.com/vekatze/sero/raw/main/archive/0-1-33.tar.zst
```

## Types

### Main Definitions

```neut
// Specifies how to encode/decode values
data sero(a) {
| Sero(
    put: (k: &put-kit, value: &a) -> unit,
    get: (k: &get-kit) -> ?meta a,
  )
}

// Encodes a value into a binary value.
define encode<a>(m: sero(a), value: &a, buffer-size: int): binary

// Decodes a value from a binary value.
define decode<a>(m: sero(a), bytes: binary): ?a
```

### Utility Functions

```neut
// Encodes a value into a binary and saves it to a file at `path`.
define encode-file<a>(m: sero(a), value: &a, buffer-size: int, path: &text): system(unit)

// Decodes a value from the file at `path`.
define decode-file<a>(m: sero(a), path: &text): system(?a)
```

### Instances

```neut
inline binary-sero: sero(binary)

inline bool-sero: sero(bool)

inline float16-sero: sero(float16)

inline float32-sero: sero(float32)

inline float64-sero: sero(float64)

inline int8-sero: sero(int8)

inline int16-sero: sero(int16)

inline int32-sero: sero(int32)

inline int64-sero: sero(int64)

inline rune-sero: sero(rune)

inline text-sero: sero(text)

inline either-sero<a, b>(!m1: sero(a), !m2: sero(b)): sero(either(a, b))

inline pair-sero<a, b>(!m1: sero(a), !m2: sero(b)): sero(pair(a, b))

inline list-sero<a>(!m: sero(a)): sero(list(a))

inline vector-sero<a>(!m: sero(a)): sero(vector(a))
```

## Example

```neut
inline _list-int-sero: sero(list(int)) {
  list-sero(int64-sero)
}

define zen(): unit {
  pin value: list(int) = [1, 2, 3, 42342, 5, 6];
  let path = "test.bin";
  let encode-result = encode-file(_list-int-sero, value, 10, path);
  match encode-result {
  | Left(e) =>
    // (failed to encode)
  | Right(_) =>
    let v = decode-file(_list-int-sero, path);
    match v {
    | Left(e) =>
      // (failed to read the file)
    | Right(v) =>
      match v {
      | Left(_) =>
        // (failed to decode)
      | Right(xs) =>
        for(xs, function (x) {
          print("right-right: ");
          print-int-line(x)
        })
      }
    }
  }
}

```
