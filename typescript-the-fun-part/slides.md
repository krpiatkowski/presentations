---
theme: apple-basic
class: 'text-center'
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
css: unocss
layout: center
---

# Typescript

The fun part

🤗


---
layout: center
---

# What is Typescript?
<v-clicks>

JavaScript with syntax for types (https://www.typescriptlang.org/)
</v-clicks>

---
layout: center
---
## So... What is Javascript?
<v-clicks>

A crazy scripting language with lots of weird facets.

```js
typeof NaN === 'number' // true
Infinity === 1/0        // true
0.1 + 0.2 === 0.3       // false
```

But... also the only really good and fast way to run code in the browser
</v-clicks>


---
layout: center
---

## Why do we need Typescript again?

<v-clicks>


Pros:
</v-clicks>
<v-clicks>

- Javascript is quite messy, typescripts, linters and transpilers can fix some of it
- DX → less error-prone development
</v-clicks>

<v-clicks>

Cons:
</v-clicks>
<v-clicks>

- Requires compiling → Slower dev-cycle
- False sense of security!?
</v-clicks>

---
layout: center
---

## Garbage in, Garbage out

<v-clicks>

```ts {1-2|3|4}
const one = JSON.parse('"1"')
const two = one++;
typeof two // number
console.info(two) // 1 
```

"1" is auto-unboxed into a `number` 😱 
</v-clicks>

---
layout: center
---

## Problem

<v-clicks>

No runtime typechecking in javascript

Typescript does **not** solve this, as types are compiled out 😭
</v-clicks>


---
layout: center
---
## How to fix?

Data → validation → casted typed → typed → ... → typed

---
layout: center
---

# Chains of trust

<v-clicks>

Data → validation → casted typed → typed → ... → typed  ✅

Static code → typed → ... → typed  ✅

NPM → 🤔 → typed? → lets go → typed → ... → typed  ✅ (💀)

typed → typed → `any` → ... → 💀

</v-clicks>


---
layout: center
---

# Typescript continued
Lets play with the type system 

---
layout: center
---

## unions
variable with multiple possible types

```ts 
let t: number | string = 'str';

t = 9;
```

---
layout: center
---

## intersections
Kinda like interfaces extending other interfaces

```ts 
interface User {
  name:string
}

interface Role {
  role: string;
}

type UserWithRole = User & Role;
```

---
layout: center
---

## Type narrowing
Helping the compiler by removing "options" from a union.

---
layout: center
---

## Control Flows

```ts{1-3|5-}
if(u /** narrowing to a specific type **/) {
  u // specific type
}

switch (u.kind /** narrowing **/){
  case 'a': 
    // type with 'a'
  case 'b':
    // type with 'b'
}

```

---
layout: center
---

## never
`never` is assignable to every type

no type is assignable to `never`

```ts
function getArea(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "square":
      return shape.sideLength ** 2;
    default:
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

---
layout: center
---

# Runtime checks
These exist in javascript - typescript can infer based on them.

---
layout: center
---

## typeof
```ts{1|3-5}
const t:any = 's'

if(typeof t === 'string') {
  t.includes('s') // this compiles because typescript infered `t` to a string
}
```

---
layout: center
---

## instanceof

```ts{1-4|6|7-}
class SomeClass {
    public someMethod() {
    }
}

const a:any = new SomeClass();

if(a instanceof SomeClass) {
    a.someMethod()
}
```


---
layout: center
---

# 🦆 Duck-typed 🦆

Typescript decided to go for duck-typing
(also called structually typed)

If it walks as a `int` and it talk like an `int` its an `int`.

---
layout: center
---


# 🦆 Duck-typed - Example 🦆

```ts
interface User {
  name: string;
  email: string;
}

// This is okey
const user:User = {
  name: 'user',
  email: 'user@user.com'
} 

// This is not!
const user:User = {
  name: 'user',
} 
```

---
layout: center
---

# Structual narrowing
We do not know what we have, so lets examin the structure

---
layout: center
---
## type predicates
```ts
const isUser = (u: any): u is User => u?.name === 'string'
```
Be careful here, remember false sense of security

---
layout: center
---
## `in` narrowing
```ts
const u:User | boolean = {name: 'user'}
if('name' in u) {
    console.info(u.name)
}
```

---
layout: center
---

# Literals
Borrowed from functional languages - quite unique feature

---
layout: center
---

## String literals
```ts
interface User {
  name: string;
  role: 'manager' | 'developer' | 'designer'
}

const u = {name: 'user', role:'manager'}
```

---
layout: center
---

## Number literals
```ts
interface HTTPResponse {
  status: 200|401|402 //...
}

```

---
layout: center
---
## Why not enums

Literals generate less code and they are easier to type

```ts
{name: 'user', role: 'manager'}
// vs
{name: 'user', role: Role.manager}
```

---
layout: center
---
## Marker Properties

```ts{1-3|5-7|9|10-13|15-17|19|21-23}
interface Manager {
  role:'manager', directReports: Developer[]
}
  
interface Developer {
    role:'developer', languages: Array<'typescript' | 'java'>, //NOT ['typescript'|'java']
}

type User = Manager | Developer;

const d:Developer = {
    role:'developer', languages: ['typescript']
}

const m: Manager = {
    role:'manager', directReports: [d]
}

const users: User[] = [d, m]

if(users[0].role === 'manager') {
    users[0].directReports //this actually work
}
```
---
layout: center
---

## as const

```ts
const d = {
    name: 'u',
    role: 'developer'
} as const 

interface Developer {
    name: string, role:'developer'
}

const newD:Developer = d; //if `as const` is missing, this will error
```

---
layout: center
---

## Distributive types
When working on unions can generate all permutations

---
layout: center
---

## Template Literal Types
```ts
type lang = 'en' | 'dk';
type keys = 'button' | 'title'

type translationKeys = `${lang}_${keys}`
// type translationKeys = "en_button" | "en_title" | "dk_button" | "dk_title"
```
---
layout: center
---

# Mapped types
Nice when working with external API (im looking at you apollo/contentstack combo)

---
layout: center
---
```ts
interface ExternallyDefinedUser {
  roles: 'developer' | 'manager'
}

interface User {
  roles: ExternallyDefinedUser['roles']
}
```
---
layout: center
---

## typeof keyof and number

```ts{1|3|5|7}
const user = {name: 'user', role:'developer', languages: ['javascript', 'java']} as const

type User = typeof user; // {name: 'user', role:'developer'} <-- this is a type!!

type Props = keyof User // keyof user or 'languages' | 'name | 'role'

type Languages = User['languages'][number] // 'javascript' | 'java'
```
---
layout: center
---

# Generics
Just like C# and Java#, typescript supports generic

The syntax is a bit special 🤪

Creating "problems" to solve is hard, but looking for them in code is easy

---
layout: center
---

## Fun with generic

```ts{1|1-6|8}
const user = {name: 'user'};

const getProp = <T, K extends keyof T> (t:T, key: K): T[K] => {
    if(typeof t === 'string') {
        t[key] + "!"
    }
    return t[key]
}

const r = getProp(user, 'name'); // 'name!', only the 'name' string literal is allowed as second parameter
```

---
layout: center
---

## Utility types

https://www.typescriptlang.org/docs/handbook/utility-types.html

20 or so utility generics, which can make things easier

of those, these are used more often:

- Partial / Require
- Extract (from union, can pattern match with props)
- Parameters (of functions, use to co-depend on other type)
- ReturnType (of functions, use to co-depend on other type)

---
layout: center
---

# Now in react

Lets have a look at `CrayonButton`

---
layout: center
---


# Conclusion
<v-clicks>

Typescript has evolved quite alot over the last few years.

I often get supprised how much it can `infer`.

Unless you activly start using the type-system, 
your never going to get good.

Read the changelogs of typescript

Remember `Data !== Type`
</v-clicks>

---
layout: end
---

