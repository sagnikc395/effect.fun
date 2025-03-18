## What is an Effect 
- something brought about by a cause or agent 
- side effects ?
- Effect type -> core of the entire effect framework.

(from the Effect website)
- an Effect is a description of a program that is lazy and immutable.
- Effect is an inmemory-type.


## what is a program ?
- series of steps and mutations.

## laziness 
- delay work until the value is really needed.

## immutability
- scripts are really immutable -> not mutated in runtime when run by the interpreter.

## functions !
- somehting which exist in js, something which is lazy -> nothing happens, even if we declare it, immutability -> not really a way to mutate the body of the function.

## effects 
- effects builds on what functions are able to offer.
- what does it offer ?
```ts 
type Function<Args,T> = (...args: Args) => T;
```
does not prescribe any error type within it, we need to pass try catch , but then we will have to handle all such possible cases.

can make it safer , by passing the error type in type signature.
```ts 
type SaferFunction<Args,T,E> = (...args: Args) => T|E ;
```
problems with this:
1. harder to discriminate the errors.
- discriminating using instanceof
```ts 
declare function Foo(): "baz" | "bar" | Error;
const result = Foo();
const error = result instanceof Error;
``` 
- discriminating using != 
```ts 
declare function Foo2(): "baz" | "bar";
const result2 = Foo2();
const error = result2 !== "baz";
```

2. Composing functions that error and creating a outer and inner function 
```ts 

declare function Inner(): "baz"| "bar" | Error;

function Outer(): number | Error {
    const result = Inner();
    if(result instanceof Error){
        return result;
    }

    return result.length
}
```
- limitation of functions that return Union as we have to manually check - forced to check Errors manually at each point.

## Effects are differnt 
- Effect have a error type parameter.

```ts 
type Effect<Value,Error= never> = /**/;

declare const foo: Effect<number,never>;
//either succeed with number and never fail

declare const bar: Effect<number,Error>;
//succeed with number or fail with a Error class

```
- we are encoding the type of errors thrughout our programs.

- Dependencies and using effect 

- in traditional ts, we are used to have to define explicty the dependency that we require.
```ts 

import {db} from './db';

function getUser(id: number): User {
    return db.users.getById(id);
}
```
- but ideally we would want the dependency scoped to the function parameter itself:
```ts 
function getUser(db: Database, id: number): User {
    return db.users.getById(id);
}
```
- benefits for this we can mock the db service itself -> in-memory or real db itself.
- this is something that is beneficial to our code.
- 3rd requirements type parameter -> types of services that Effect require.
```ts 
type Effect<Value,Error= never,Requirement = never> = /* */;
```

- so consider a real world scenario where we are interacting with db , and our function looks like this 

```ts 
function updateEmail(db: Database,logger: Logger, telemetry: Telemetry,id: number, newEmail: string): User {
    const user = db.users.getById(id);
    db.users.updateEmail(id,newEmail);
    logger.info(`Update email for user ${id}`);
    telemetry.record(`email updated: `, { id});
    return user;
}
```
- we can then write a Effect type simply that can replace the long function parameter
```ts 
declare const getUser: Effect<User,NotFoundError,Database>;
```
- on a type level, we shouldnt be able to execute the functions when this third type parameter of Requirement is not never - we depend on some service that we dont have and it shouldnt affect it.

- similarly we can write a email that we have , requiring multiple services:
```ts 

declare const updateEmail: Effect <User,NotFoundError,Database| Logger | Telemetry>;
```

- Effects also abstracts over sync vs async:
 - in traditional js/ts we have the function coloring problem -> we can only call async functions from other async functions and sync functions from other sync functions.
- Effects abstracts over this.
```ts 
//tradtional ts 
declare async function getUser(id: number): Promise<User>;
```
- using effect:
```ts 

declare function getUser(id: number): Effect<User,NotFoundError,UserRepo>;
```

