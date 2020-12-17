# Naming & Grouping
> The identity functor, and explains that it takes a value and returns a value. The identity functor is founded on category theory, which states that functional programming necessitates both composition and identity.


## 😞 Object Oriented 
```js
class User {
	constructor(first, last) {
		this.first = first
		this.last = last
	}
	
	full() {
		return this.first + ' ' + this.last
	}
}

const user = new User('Jordan', 'Gilliam')
user.full() // Jordan Gilliam
```

## Functional 

🤔
```js
const user = {first: 'Jordan', last: 'Gilliam'}
const full = (first, last) => [first, last].join(' ')

full(user.first, user.last) // Jordan Gilliam
```

😌
```js
const user = {first: 'Jordan', last: 'Gilliam'}

// associative + composition
const joinWithSpace = ...args => args.join(' ')

joinWithSpace(user.first, user.last)

joinWithSpace('a', 'b', 'c') // 'a b c'

joinWithSpace(joinWithSpace('a', 'b'), 'c') // 'a b c'

joinWithSpace('a', joinWithSpace('b', 'c')) // 'a b c'
```

🔥 
```js
// using joinable as an interface :)
// ENCAPSULATION -> ALL WE CAN DO IS CALL JOIN
const joinWithSpaces = joinable => joinable.join(' ')


joinWithSpaces([user.first, user.last]) // Jordan Gilliam
```

🧙‍♂️ 
**Encapsulation**  is the dual of **Abstraction**
> in this case we don't know anything about a so we can't do anything with it
```js
// ENCAPSULATION -> ALL WE CAN DO IS RETURN A


const identity = a => a
```

## Guiding Principle
### **Highly Generalized Functions** > Highly Derivative Functions

#js #functional-programming #architecture #pattern