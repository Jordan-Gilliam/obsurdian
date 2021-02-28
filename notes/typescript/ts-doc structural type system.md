# Structural Type System

> One of TypeScript’s core principles is that type checking focuses on the shape that values have. 

- This is sometimes called “duck typing” or “structural typing”.

### In a structural type system, if two objects have the same shape, they are considered to be of the same type.

```ts
interface Point {
	x: number;
	y: number;
}

function printPoint {
	console.log(`${p.x}, ${p.y}`);
}

const point = { x: 3, y: 9 };
printPoint(point); // "3, 9"
```

###  The point variable is never declared to be a Point type.
	- However, TypeScript compares the shape of point to the shape of Point in the type-check.
	- They have the same shape, so the code passes!

### The shape-matching only requires a subset of the object’s fields to match.
```ts

✅
const point3 = { x: 12, y: 26, z: 89 };
printPoint(point3); // prints "12, 26"

✅
const rect = { x: 33, y: 3, width: 30, height: 80 };
printPoint(rect); // prints "33, 3"

🛑
const color = { hex: "#187ABF" };
printPoint(color);

ERROR: Argument of type '{ hex: string; }' is not assignable to parameter of type 'Point'.
  Type '{ hex: string; }' is missing the following properties from type 'Point': x, y
```

## MAIN POINT 
**If the object or class has all the required properties, TypeScript will say they match, regardless of the implementation details.**