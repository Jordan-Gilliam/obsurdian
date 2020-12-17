### variables

> In Rust, variables are immutable by default.

`let mut food = String::new();`

- creates a mutable variable called **food**
- sets **food** variable to a **new string instance** [UTF-8 encoded bit of text]

- **The `::` syntax** in the `::new` line indicates that `new` is an _associated function_ of the `String` type. An associated function is implemented on a type, in this case `String`, rather than on a particular instance of a `String`. Some languages call this a **_static method_.**

#rust