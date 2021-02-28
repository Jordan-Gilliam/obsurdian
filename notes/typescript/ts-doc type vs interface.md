# Interface's vs Types
#### There are **two** main tools tp declare the shape of an object
	- interfaces
	- type aliases
	
> They are very similar and for the most common cases act the same

<br>

### What do I use?
1. You should prefer `interface`.  specifically, because you will get better error messages. 

2. Use `type` when you need specific features.

```ts
type BirdType = {
  wings: 2;
};

interface BirdInterface {
  wings: 2;
}
```

### Because TypeScript is a structural type system, it's possible to intermix their use too

```ts
const bird1: BirdType = { wings: 2 };
const bird2: BirdInterface = { wings: 2 };

const bird3: BirdInterface = bird1;
```

### They both support extending other interfaces and types
-  Type aliases do this via intersection types
-  interfaces have a keyword

```ts
type Owl = { nocturnal: true } & BirdType;
type Robin = { nocturnal: false } & BirdInterface;

interface Peacock extends BirdType {
  colourful: true;
  flies: false;
}
interface Chicken extends BirdInterface {
  colourful: false;
  flies: false;
}

let owl: Owl = { wings: 2, nocturnal: true };
let chicken: Chicken = { wings: 2, colourful: false, flies: false };
```

### One major difference between type aliases vs interfaces are that interfaces are open and type aliases are closed
> This means you can extend an interface by declaring it a second time

```ts
interface Kitten {
  purrs: boolean;
}

interface Kitten {
  colour: string;
}
```

> In the other case a type cannot be changed outside of it's declaration.

```ts
type Puppy = {
  color: string;
};

type Puppy = {
  toys: number;
};
```


#### Depending on your goals, this difference could be a positive or a negative. However for publicly exposed types, it's a better call to make them an interface

- [	One of the best resources for seeing all of the edge
	 cases around types vs interfaces, this stack overflow
	 thread is a good place to start](https://stackoverflow.com/questions/37233735/typescript-interfaces-vs-types/52682220#52682220)