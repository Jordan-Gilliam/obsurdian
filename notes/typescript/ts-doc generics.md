# Generics
> Generics provide variables to types. A common example is an array. An array without generics could contain anything. An array with generics can describe the values that the array contains.

```ts
type StringArray = Array<string>;
type NumberArray = Array<number>;
type ObjectWithNameArray = Array<{ name: string }>;
```

### You can declare your own types that use generics
```ts
interface HikingBackpack<Type> {
	add: (obj: Type) => void;
	get: () => Type;
}

/** 
	This line is a shortcut to tell TypeScript there is a
	constant called `hikingBackpack`, and to not worry about where it came from.
**/
declare const hikingBackpack: Backpack<string>;


// object is a string, because we declared it above as the variable part of HikingBackpack.
const object = backpack.get();

// Since the backpack variable is a string, you can't pass a number to the add function.
backpack.add(23);
ERROR: Argument of type 'number' is not assignable to parameter of type 'string'.
```

