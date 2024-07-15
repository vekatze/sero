# marshal

`marshal` is a [Neut](https://vekatze.github.io/neut/) module that saves values to files and loads them from there.

## Summary

```neut
data marshal(a) {
| Marshal(
    encode: (buffer: &builder, value: &a) -> unit,
    decode: (bytes: &binary, cursor: &cell(int)) -> ?a,
    necess: (a) -> meta a,
  )
}

define save<a>(m: marshal(a), value: &a, path: &text, buffer-size: int): system(unit)

define load<a>(m: marshal(a), path: &text): system(?a)

constant this.class.int64.as-marshal: marshal(int64)

constant this.class.float64.as-marshal: marshal(int64)

constant this.class.text.as-marshal: marshal(text)

...

inline this.class.list.as-marshal<a>(!m: marshal(a)): marshal(list(a))

inline this.class.either.as-marshal<a, b>(!m1: marshal(a), !m2: marshal(b)): marshal(either(a, b))

inline this.class.pair.as-marshal<a, b>(!m1: marshal(a), !m2: marshal(b)): marshal(pair(a, b))

...
```

## Example

```neut
constant marshal-list-int: marshal(list(int)) {
  this.class.list.as-marshal(this.class.int64.as-marshal)
}

// save and load a list of integers
define zen(): unit {
  pin value: list(int) = [1, 2, 3, 4, 5, 6] in
  let path = "test.bin" in
  let _ = save(marshal-list-int, value, path, 10) in
  let v = load(marshal-list-int, path) in
  match v {
  | Left(e) =>
    printf("error: {}\n", [get-error-message(e)]);
  | Right(v) =>
    match v {
    | Left(_) =>
      Unit
    | Right(xs) =>
      for(xs, function (x) {
        printf("{}\n", [show-int(x)])
      })
    }
  }
}

```
