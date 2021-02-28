# Unions
```ts
type IsDaytime = true | false
```

### A popular use-case for union types is to describe the set of strings or numbers literal that a value is allowed to be
> think state machines :)


```ts
type FetchStates = 'idle' | 'loading' | 'error'
type ToggleStates = 'initial' | open' | 'closed'
```

### Unions provide a way to handle different types too!
> For example, a function that takes an array or a string

```ts
function getLength(obj: string | string[]) {
  return obj.length;
}
```

### To learn the type of a variable, use `typeof`
> For example, fn returns different values depending whether it is passed a string or array

```ts
function wrapInArray(obj: string | string[]) {
  if (typeof obj === "string") {
    return [obj];
//          ^ = (parameter) obj: string
  } else {
    return obj;
  }
}
```