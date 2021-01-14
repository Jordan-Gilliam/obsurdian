# tuple types
Makes composing multiple functions/pipes much easier

```ts
type Foo<T extends any[]> = [boolean, ...T, boolean];
```

## labeled tuple types
```ts
type Address = [
	streetNumber: number,
	city: string,
	state: string,
	postal: number,
];

function printAddress(...address: Address) {}
printAddress()
```

