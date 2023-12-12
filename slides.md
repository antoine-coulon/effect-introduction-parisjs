---
# try also 'default' to start simple
theme: default
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://images.unsplash.com/photo-1533134486753-c833f0ed4866?q=80&w=2070&auto=format&fit=crop&ixlib=rb-4.0.3&ixid=M3wxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8fA%3D%3D
# apply any windi css classes to the current slide
class: 'text-center'
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: false
# some information about the slides, markdown enabled
info: |
  ## Antoine Coulon, Paris TypeScript # 35
  
  Effect, une solution efficace aux problèmes de software engineering
# persist drawings in exports and build
drawings:
  persist: false
# use UnoCSS (experimental)
css: unocss
---

## Effect, une solution efficace aux problèmes de software engineering

<br>

<h4 class="mt-10">
  Antoine Coulon @ Paris TypeScript #35 - 12/12/2023
</h4>

---

<div class="grid grid-cols-10 gap-x-4 pt-5 pr-10 pl-10">

<div class="col-start-1 col-span-7 grid grid-cols-[3fr,2fr] mr-10">
  <div class="pb-4">
    <h1>Antoine Coulon</h1>
    <div class="leading-8">
      Lead Software Engineer @ evryg
      <br>
      Open Source: NodeSecure, skott, Effect, Rush.js
    </div>
  </div>
  <div class="border-l border-gray-400 border-opacity-25 !all:leading-12 !all:list-none my-auto">
  </div>

  <div>
    <div class="mb-4"><ri-github-line color="blue"/> <b color="opacity-30">antoine-coulon</b></div>
    <div class="mb-4"><ri-twitter-line color="blue"/> <b color="opacity-30">c9antoine</b></div>
    <div class="mb-4"><ri-user-3-line color="blue"/> <b color="opacity-30">dev.to/antoinecoulon</b></div>
  </div>
</div>

<div class="pl-20 col-start-8 col-span-10">
  <img src="https://avatars.githubusercontent.com/u/43391199?s=400&u=b394996dd7ddc0bf7a317185ee9c378d5b609e12&v=4" class="rounded-full w-40" />
</div>

</div>


<style>
h1 {
  color: #008ad6;
  font-weight: bold;
}
</style>


---

## Effect : le <b color="green">pourquoi</b> plutôt que le <b color="orange">comment</b>

<br>

- Comprendre quels sont les problèmes les plus communs auxquels nous faisons face en tant que software engineers

- Prendre connaissances des limites actuelles quand on utilise TypeScript et les runtimes JavaScript

- Apprentissage basique d'Effect

---

## Effect ne résout pas uniquement des problèmes propres à TypeScript 

<div class="grid grid-cols-2 gap-x-4 pt-5">

<div class="pt-3 ml-5">
<ul>
<li> Explicitation </li>
<li> Testing </li>
<li> Résilience </li>
<li> Concurrence </li>
<li> Gestion des ressources </li>
</ul>
</div>

<div class="pt-3">
<ul>
<li> Type-Safety </li>
<li> Composabilité </li>
<li> Maintenabilité </li>
<li> Efficacité & Performance <ri-hourglass2-fill color="blue"/> </li>
<li> Monitoring <ri-hourglass2-fill color="blue"/> </li>
</ul>
</div>

</div>
---

## Prélude : qu'est-ce qu'un Effect

<br>

Un Effect se définit à l'aide d'un datatype <b color="blue">Effect<R, E, A></b>
- <b color="blue">[R]</b> représente l'ensemble des dépendances requises pour que l'Effect puisse être exécuté
- <b color="blue">[E]</b> représente l'ensemble des erreurs connues qui peuvent survenir lors de l'exécution
- <b color="blue">[A]</b> représente le résultat qui peut être produit en cas de réussite de l'exécution

<br>

```ts
import { Effect } from "effect";  

const randomString: Effect.Effect<never, never, string> = //

const now: Effect.Effect<Clock, never, Date> = //
```

---

## Explicitation : mais où sont les erreurs et dépendances ?

<br>

> La capacité à rendre un programme transparent, facile à comprendre et sans laisser de place à une quelconque ambiguité vis-à-vis de son fonctionnement.

<div class="flex justify-center mt-3">
<img width="500" src="/whereis.gif" />
</div>

---

## Explicitation (erreurs) : opérations synchrones

<div class="grid grid-cols-2 gap-x-4 pt-5">

<div>

```ts
// random.ts
export function generateRandomNumber(): number {
  // some implementation...
}
```

```ts
// main.ts
import { generateRandomNumber } from "./random";

function main() {
  return generateRandomNumber() * 10;
}
```
</div>

<div v-click>

```bash
❯ node main.js
Error: Oops!
    at generateRandomNumber
```

<img width="500" src="/uncaught-exception.gif" />

</div>

</div>

---

## Explicitation (erreurs) : à la recherche de l'info perdue

<div>

```ts
// main.ts

function isSomeErrorException(exception: unknown): exception is SomeError {
  return exception instanceof Error && exception.name === "SomeError";
}

function isSomeOtherErrorException(exception: unknown): exception is SomeOtherError {
  return exception instanceof Error && exception.name === "SomeOtherError";
}

try {
  generateRandomNumber();
} catch (exception: unknown) {
  // Worst case? We don't even know what to expect from "exception"

  // Best case:
  if (isSomeErrorException(exception)) {
    // do something
  } else if (isSomeOtherErrorException(exception)) {
    // do something else
  }
}
```

</div>


---

## Explicitation (erreurs) : opérations asynchrones

Utilisation des Promises, une des primitives asynchrones

```ts
interface Promise<T> {}

async function generateRandomNumber(): Promise<number> {
  //
}

generateRandomNumber().catch((_: any) => {});

try {
  await generateRandomNumber();
} catch (exception: unknown) {}
```

<br>

- On fait face aux mêmes problèmes
- Divergences/différences au niveau des APIs

---

## Explicitation (erreurs) : Effect à la rescousse

```ts {3-9|11-12|14-21|11-23} {lines:true}
import { Effect, pipe } from "effect";

class NumberIsTooBigError {
  readonly _tag = "NumberIsTooBigError";
}

class NumberIsTooSmallError {
  readonly _tag = "NumberIsTooSmallError";
}

export const generateRandomNumber: Effect.Effect<
never, NumberIsTooBigError | NumberIsTooSmallError, number> = //

const main = pipe(
  generateRandomNumber,
  // exhaustive pattern matching
  Effect.catchTags({
    NumberIsTooBigError: () => Effect.succeed(0),
    NumberIsTooSmallError: () => Effect.succeed(1),
  })
);

type main = Effect.Effect<never, never, number>;
```

---

## Explicitation : et les dépendances ?

<b>Gestion explicite et puissante des erreurs typées</b>
- filtering
- recovery
- mapping
- pattern matching

<br>

<div>

```ts {0|3-4|2|all} 
interface Effect<
  R, // context channel
  E, // error channel 
  A  // success channel
> {}
```
</div>

---

## Explicitation (dépendances) : inférence automatique des dépendances

```ts {9-13|19-21|16|all} {lines:true}
import { Context, Effect } from "effect";

class UserAlreadyExistsError {
  readonly _tag = "UserAlreadyExistsError";
}

class CreatedUser {}

interface UserRepository {
  createUser: () => Effect.Effect<never, UserAlreadyExistsError, CreatedUser>;
}

const UserRepository = Context.Tag<UserRepository>();

const registerUser: Effect.Effect<
  UserRepository, 
  UserAlreadyExistsError, 
  CreatedUser
> = Effect.flatMap(UserRepository, (userRepository) => userRepository.createUser());
```

<!-- 

Dès lors qu'une dépendance est utilisée elle est automatiquement propagée et inférée dans le premier générique.

-->

---

## Explicitation (dépendances) : type-safety autour du contexte


```ts {9-11|3-4|13-18|21} {lines:true}
import { Effect, pipe } from "effect";

const registerUser: Effect.Effect<
  UserRepository, 
  UserAlreadyExistsError,
  CreatedUser
> = // whatever Effect there

// ts(2345): Type 'UserRepository' is not assignable to type 'never'
//             ^^^^^^^^^^^^
Effect.runSync(registerUser);

const registerUserWithSatisfiedDependencies: Effect<never, UserAlreadyExistsError, CreatedUser> = pipe(
  registerUser,
  Effect.provideService(UserRepository, {
    createUser: () => Effect.succeed(new CreatedUser()),
  })
);

// compiles and works
Effect.runSync(registerUserWithSatisfiedDependencies);
```

<!-- 

Alors oui le côté explicite est utile pour la compréhension mais il apporte surtout de la type-safety vis à vis du compiler.
Tant que R n'est pas "never", le programme ne compilera pas. On doit donc passer par de la DI qui est type-safe.

R est inféré de manière profonde -> généraliser avec l'ensemble d'un programme

-->


---

## Testing

<br>

> Testing c'est l'art de garantir que le système produit les résultats attendus.

Effect, au service du <b color="blue">Dependency Inversion Principle</b>

```ts {1-5|8,12|all} {lines:true}
class InMemoryUserRepository implements UserRepository {
  createUser() {
    //
  }
}

test("Should blabla", async () => {
  const fakeRepository = new InMemoryUserRepository();

  const user = await pipe(
    createUser(), 
    Effect.provideService(UserRepository, fakeRepository),
    Effect.runPromise
  );

  expect(user).toEqual("whatever");
});
```

<!--

Question: à quelle lettre correspond le DIP dans Solid ?

Explication du DIP et de ce que ça apporte, on casse les "hidden dependencies"

Effect facilite la testabilité

-->
---

## Résilience

<br>

> L'art de designer et d'implémenter des systèmes qui peuvent réagir à des erreurs attendues et de les gérer correctement.

<br>

- Recovery
- Retries
- Interruptions
- Resource management
- Circuit breakers
- etc

---

## Résilience : recovery et retries

<div class="grid grid-cols-5 gap-x-4 pt-5">

<div class="col-start-1 col-span-2" v-click> 

```ts
// RECOVERY

import { Effect, pipe } from "effect";

const program = () => pipe(
  Effect.fail(new Error()),
  // Recover from all errors
  Effect.catchAll((_) => Effect.succeed(0)),
  // Recover from all errors and provide specific behavior per error
  Effect.catchTags({
    _1: () => Effect.succeed(1),
    _2: () => Effect.succeed(2)
  }),
  // Recover from specific errors only
  Effect.catchTag({
    _1: () => Effect.succeed(1)
  })
)
```

</div>

<div class="col-start-3 col-span-5" v-click>

```ts
// RETRY

import { Effect, pipe, Schedule, Duration } from "effect";

const schedulePolicy = pipe(
  Schedule.recurs(5),
  Schedule.addDelay(() => Duration.millis(500)),
  Schedule.compose(Schedule.elapsed),
  Schedule.whileOutput(Duration.lessThanOrEqualTo(Duration.seconds(3))),
  Schedule.whileInput(
    (error) => error instanceof Error && error.message !== "_"
  )
);

const programWithRetryPolicy = pipe(
  Effect.failSync(() => new Error("Some_error")),
  Effect.retry(schedulePolicy),
  Effect.catchAll(() => Effect.sync(() => {
    console.log("Program ended")
  }))
);
```

</div>
</div>

---

## Résilience : la nécessité d'interruption et de cleanup

<div class="grid grid-cols-2 gap-x-4 pt-5">

<div>

```ts
import { setTimeout } from "node:timers/promises";

const leakingRace = () => Promise.race([
  setTimeout(1000), 
  setTimeout(10000)
]);

function raceWithInterruptions() {
  const abortController1 = new AbortController();
  const abortController2 = new AbortController();

  async function cancellableTimeout1() {
    await setTimeout(1000, undefined, { signal: abortController1.signal });
    abortController2.abort();
  }

  async function cancellableTimeout2() {
    await setTimeout(10000, undefined, { signal: abortController2.signal });
    abortController1.abort();
  }

  return Promise.race([cancellableTimeout1(), cancellableTimeout2()]);
}
```
</div>

<div v-click>

```ts
import { Effect } from "effect";

const race = [
  Effect.sleep(1000),
  Effect.sleep(10_000).pipe(
    Effect.onInterrupt(() => Effect.log("interrupted"))
  ),
];
```

<br>

```ts
const backgroundJob = Effect.async(() => {
  const timer = setInterval(() => {
    console.log("processing job...");
  }, 500);

  return Effect.sync(() => {
    console.log("releasing resources...");
    clearInterval(timer);
  });
});
```


</div>


</div>

---

## Concurrence 

<br>

> L'art de gérer des opérations en simultanée ou de manière coopérative avec un contrôle sur leur exécution et des garanties fortes sur l'intégrité de l'état du programme, afin d'améliorer son efficacité.

<b color="blue">"concurrency is about dealing with lots of things at once"</b>, Rob Pike

Gérer la concurrence correctement c'est compliqué :

- Difficile d'avoir un modèle d'exécution déterministe 
- Problème des ressources partagées + gestion des ressources
- Deadlocks, race conditions
- Contrôle et efficacité Mémoire et CPU 
- ...etc

---

## Concurrence : bounded vs unbounded

<div class="grid grid-cols-2 gap-x-4 pt-5">

<div v-click>

```ts
// Unbounded 
const userIds = Array.from(
  { length: 1000 }, (_, idx) => idx
);

function fetchUser(id: number): Promise<User> {}

function retrieveAllUsers() {
  return Promise.all(userIds.map(fetchUser));
}
```

- Pas de gestion des interruptions
- Pas de garantie sur la libération des ressources
- Pas de contrôle sur l'exécution concurrente
- `Promise.allSettled` permet un contrôle plus fin sur le résultat produit mais souffre des mêmes problèmes

<!-- <div class="grid justify-left">
  <img width="400" src="/unbounded.gif">
</div> -->

</div>

<div v-click>

```ts
const users = pipe( 
  userIds,
  Effect.forEach(
    (id) => Effect.promise(() => fetchUser(id)), 
    { concurrency: 30 },
    // OR
    { concurrency: "unbounded" },
    // OR
    { concurrency: "inherited" }
  )
);
```

</div>
</div>

<!-- 

Transition parfaite vers ressource management

-->
---

## Resource management

<br>

> L'art de gestion du cycle de vie des ressources allouées lors de l'exécution du programme

<br>

- Proposal "Explicit Resource Management", mais généralisé et plus composable
- Introduction de Scopes, dès lors qu'on a plus besoin des ressources, des finalizers sont appelés
- Finalizers appelés dès lors qu'une interruption/erreur d'un Effect est produite
- Contexte qui contrôle la propagation des scopes, on évite le "props drilling" des Abort Signals 
- Libération Sync/Async, peut être elle-même rendue interruptible/non-interruptible

---

## Et aussi d'autres modules...


<div class="grid grid-cols-2 gap-x-4 pt-5">

<ul>
<li>Layer</li>
<li>Stream</li>
<li>Queue</li>
<li>Semaphore </li>
<li>Pub/Sub </li>
<li>Batching/Caching</li>
<li>Tracing/Monitoring</li>
<li>Config</li>
</ul>

<ul>
<li>@effect/schema</li>
<li>@effect/cli</li>
<li>@effect/fastify</li>
<li>@effect/http</li>
<li>@effect/rpc</li>
<li>@effect/opentelemetry</li>
</ul>

</div>

---

## Merci d'avoir écouté !

- **Effect website**: <b color="cyan">https://effect.website</b>
  
- **Effect introduction**: <b color="cyan">https://github.com/antoine-coulon/effect-introduction</b>


<br>

### Questions ?
