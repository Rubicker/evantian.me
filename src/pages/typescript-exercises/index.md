---

title: TypeScript Exercises

date: '2020-05-04'

spoiler: Test how good you are at TypeScript

---

Recently, I found a [repository](https://github.com/mdevils/typescript-exercises) named typescript-exercises and walk through the exercises it offers, they're pretty useful for me. So I'll record here my solutions and the things I learnt from.

## exercise-00

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-00/index.ts)

This one is pretty easy, all we need to do is defining an interface `User`:

```ts{1-5}
interface User {
  name: string;
  age: number;
  occupation: string;
}

const users: User[] = [
  // ...
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

Union types is a very powerful feature. Consider this:

```ts
function isAdmin(person: User | Admin): boolean {
  return role in Person
}

if (isAdmin(somePerson)) {
  // error, TypeScript doesn't know if somePerson has a property named role
  somePerson.role
}

```
With union type and user-defined type guard:
```ts
function isAdmin(person: User | Admin): person is Admin {
  return 'role' in person
}

if (isAdmin(somePerson)) {
  // here we can get role directly by somePerson.role
  // because TypeScript help us narrow down somePerson's type => Admin
}
```

## exercise-02

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-02/index.ts)

This one we need to fix logPerson by letting TypeScript know what type it is before access it's properties:

```ts{3}
function logPerson(person: Person) {
  let additionalInformation: string;
  if ('role' in person) {
    additionalInformation = person.role;
  } else {
    additionalInformation = person.occupation;
  }
  // ...
}
```

## exercise-03

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-03/index.ts)

Yup, this time user-defined type guard is gonna show up:

```ts{17, 21}
interface User {
  type: 'user';
  name: string;
  age: number;
  occupation: string;
}

interface Admin {
  type: 'admin';
  name: string;
  age: number;
  role: string;
}

type Person = User | Admin

function isUser(person: Person): person is User {
  return person.type === 'user'
}

function isAdmin(person: Person): person is Admin {
  return person.type === 'admin'
}

function logPerson(person: Person) {
  let additionalInformation: string = '';
  if (isAdmin(person)) {
      additionalInformation = person.role;
  }
  if (isUser(person)) {
      additionalInformation = person.occupation;
  }
  // ...
}
```

## exercise-04

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-04/index.ts)

This task asks us to type arguements of filterUsers, `criteria` has some values of `Users`, so `Partial<T>` is what we want:

```ts
type Partial<T> = {
  [P in key of T]?: T[P]
}
```

```ts
function filterUsers(persons: Person[], criteria: Partial<User>): User[] {
  return persons.filter(isUser).filter((user) => {
    let criteriaKeys = Object.keys(criteria) as (keyof User)[];
    return criteriaKeys.every((fieldName) => {
      return user[fieldName] === criteria[fieldName];
    });
  });
}
```

## exercise-05

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-05/index.ts)

We need [overload](https://www.typescriptlang.org/docs/handbook/functions.html#overloads) to return different types based on different arguments:

```ts
type PersonType = Admin['type'] | User['type']

function filterPersons(
  persons: Person[],
  personType: Admin['type'],
  criteria: Partial<Admin>,
): Admin[];

function filterPersons(
  persons: Person[],
  personType: User['type'],
  criteria: Partial<User>,
): User[];

function filterPersons(
  persons: Person[],
  personType: PersonType,
  criteria: Partial<Admin | User>,
) {
  return persons
    .filter((person) => person.type === personType)
    .filter((person) => {
      let criteriaKeys = Object.keys(criteria) as (keyof Person)[];
      return criteriaKeys.every((fieldName) => {
        return person[fieldName] === criteria[fieldName];
      });
    });
}

let usersOfAge23: User[] = filterPersons(persons, 'user', { age: 23 });
let adminsOfAge23: Admin[] = filterPersons(persons, 'admin', { age: 23 });
```

## exercise-06

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-06/index.ts)

Obviously, we need generics to type function `swap`:

```ts
function swap<T1, T2>(v1: T1, v2: T2): [T2, T1] {
  return [v2, v1]
}
```

## exercise-07

[source address](https://github.com/mdevils/typescript-exercises/blob/master/exercises/exercise-07/index.ts)

To type PowerUser, we need to combine Admin and User except property 'type', then add its own 'type':

```ts
type PowerUser = Omit<Admin & User, 'type'> & {
  type: 'powerUser';
}
```
