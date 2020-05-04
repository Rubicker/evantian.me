---

title: TypeScript Exercises

date: '2020-05-04'

spoiler: Test how good you are at TypeScript

---

Recently, I found a [repository](https://github.com/mdevils/typescript-exercises) named typescript-exercises and walk through the exercises it offers, they're pretty useful for me. So I'll record here my solutions and the things I learnt from.

## exercise-00

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-00/index.ts)

This one is pretty easy, all we need to do is define an interface `User` and use it:

```ts{1-5}
interface User {
  name: string;
  age: number;
  occupation: string;
}

const users: User[] = [
  {
    name: 'Max Mustermann',
    age: 25,
    occupation: 'Chimney sweep'
  },
  {
    name: 'Kate Müller',
    age: 23,
    occupation: 'Astronaut'
  }
]

function logPerson(user: User) {
  console.log(` - ${chalk.green(user.name)}, ${user.age}`)
}
```

## exercise-01
[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-01/index.ts)

Here, we need to define a union type that can be a `User` or `Admin` type:

```ts{13}
interface User {
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  name: string;
  age: number;
  role: string;
}

type Person = User | Admin

const persons: Person[] = {
  // both user and admins are here
}

function logPerson(person: Person) {
  console.log(` - ${chalk.green(person.name)}, ${person.age}`)
}
```

> [Advanced Types - union types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#union-types)