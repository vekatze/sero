# sero

`sero` is a [Neut](https://vekatze.github.io/neut/) module that saves values to files and loads them from there.

## Installation

```sh
neut get sero https://github.com/vekatze/sero/raw/main/archive/0-1-22.tar.zst
```

## Types

### Main Definitions

```neut
// Specifies how to encode/decode values
data sero(i, o) {
| Sero(
    put: (k: &put-kit, value: &i) -> unit,
    get: (k: &get-kit) -> ?meta o,
  )
}

// Encodes a value into a binary value.
define encode<i, o>(m: sero(i, o), value: &i, buffer-size: int): binary

// Decodes a value from a binary value.
define decode<i, o>(m: sero(i, o), bytes: binary): ?o
```

### Utility Functions

```neut
// Encodes a value into a binary and saves it to a file at `path`.
define encode-file<i, o>(m: sero(i, o), value: &i, buffer-size: int, path: &text): system(unit)

// Decodes a value from the file at `path`.
define decode-file<i, o>(m: sero(i, o), path: &text): system(?o)
```

### Instance Constructors

You can quickly construct values of type `sero(in, out)` by using some of the following terms:

```neut
inline binary-sero: sero(binary, binary)

inline bool-sero: sero(bool, bool)

inline float16-sero: sero(float16, float16)

inline float32-sero: sero(float32, float32)

inline float64-sero: sero(float64, float64)

inline int8-sero: sero(int8, int8)

inline int16-sero: sero(int16, int16)

inline int32-sero: sero(int32, int32)

inline int64-sero: sero(int64, int64)

inline rune-sero: sero(rune, rune)

inline text-sero: sero(text, text)

inline either-sero<i1, o1, i2, o2>(!m1: sero(i1, o1), !m2: sero(i2, o2)): sero(either(i1, i2), either(o1, o2))

inline list-sero<i, o>(!m: sero(i, o)): sero(list(i), list(o))

inline pair-sero<i1, o1, i2, o2>(!m1: sero(i1, o1), !m2: sero(i2, o2)): sero(pair(i1, i2), pair(o1, o2))

inline vector-sero<i, o>(!m: sero(i, o)): sero(vector(i), vector(o))
```

## Example

```neut
inline _list-int-sero: sero(list(int), list(int)) {
  list-sero(int64-sero)
}

define zen(): unit {
  printf("O_CREAT: {}\n", [show-int(from-c-int(O_CREAT))]);
  printf("mode: {}\n", [show-int(from-c-int(core.file.mode.interpret(default-file-mode)))]);
  pin value: list(int) = [1, 2, 3, 4, 5, 6] in
  let path = "test.bin" in
  let _ = encode-file(_list-int-sero, value, 10, path) in
  let v = decode-file(_list-int-sero, path) in
  match v {
  | Left(e) =>
    printf("left: {}\n", [get-error-message(e)])
  | Right(v) =>
    print("right\n");
    match v {
    | Left(_) =>
      print("right-left")
    | Right(xs) =>
      for(xs, function (x) {
        printf("right-right: {}\n", [show-int(x)])
      })
    }
  }
}

```
