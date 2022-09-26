# Names

After we've “blown the dust off the code,” we're ready to consider more serious problems.

The improvements and techniques we're about to discuss next are much harder to rank from simple to complex. It's not always obvious when and which technique needs to be applied.

| Chapter order 🎯                                                                                                                                                                                          |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| I suggest thinking of this and the following chapters not as a “step-by-step manual,” but rather as a list of problems and their possible solutions.                                                      |
| The chapters in the book are arranged so that the size of suggested code changes gradually grows. But there's no guarantee that the problems will occur in this order when you refactor the real project. |
| Be careful when analyzing your code. Note and write down the problems in the way you feel most comfortable with, and use this book as supporting material.                                                |

Attention to the names of variables, functions, classes, and modules can be Pandora's box in the world of refactoring. “Unclear” names can be a signal of high coupling between modules, inadequate separation of concerns, or “leaking” abstractions. In this chapter, we'll talk about how obscure names can help us find problems in our code.

## General Guidelines

An “obscure” name can hide valuable project details and the experience of other developers. It's important for us to preserve them, so it's worth diving into the meaning of such a name rather than discarding it completely. If during refactoring we see a name whose meaning isn't clear to us, we can try:

- Find someone on the team who knows the meaning of that name;
- Search for the meaning in the code documentation;
- As a last resort, do a series of experiments, changing the variable and seeing how it affects the code.

| By the way 🔬                                                                                                       |
| :------------------------------------------------------------------------------------------------------------------ |
| For the last case, reliable tests are important: they can show the result of changing variable values more clearly. |

It's better to keep the gained knowledge directly in the code by renaming a variable. If this can't be done, it's worth keeping it _as close to the code_ as possible. We can use a comment, documentation, a commit message, or a ticket description. The point is to allow developers to find and use this information _easier_.

## Too Short Names

We can consider a name good if it adequately represents the meaning of the variable or information about the domain. Too short names and uncommon abbreviations don't do that. Sooner or later such a name will be misread because all the “knowledgeable” developers will stop working on the project.

```js
// Names `d`, `c`, and `p` are too short,
// so it's hard to reason about their meaning:

let d = 0;
if (c === "HAPPY_FRIDAY") d = p * 0.2;

// Variables with full-word names
// are easier to understand:

let discount = 0;
if (coupon === "HAPPY_FRIDAY") discount = price * 0.2;
```

One-letter names are okay in very short pieces of code, like `for`-loops. But for business logic, it's better to use more descriptive names.

The same goes for abbreviations. I prefer not to use abbreviations in code except in 2 cases:

- If the abbreviation is a well-known one (e.g. USA, OOP, USD);
- If it's used in this exact form in the domain (e.g. Challenge Rate as CR in encounter calculator for D&D).

In other cases, it's better to choose full word forms for the name:

```js
// Okay, USD is a common abbreviation:
const usd = {};

// Might be okay if the domain is related to maths
// and the program works with derivatives:
const dx = 0.42;

// Not okay, should use the full form:
const ec = 0.6188;

// Much better:
const elasticityCoefficient = 0.6188;
```

## Too Long Names

Too long names signal that the entity is doing too many different things. The key word here is _“different”_ because non-cohesive functionality is the most difficult to combine in a single name.

When functionality is non-cohesive, the name tries to convey all its work details in a single phrase. This bloats the name and makes it noisy. We should pay attention to names that contain words like `that`, `which`, `after`, etc.

Most often long names are a signal of a function that does too much. Such a function most likely uses terms that are either too primitive for it, or too abstract, and it can't find the right words to express its meaning. The main heuristic for finding such functions is to try to come up with a shorter name for the function. If we can't do that, the function is probably doing too much.

```js
async function submitOrderCreationFormIfValid() {
  // ...
}
```

The `submitOrderCreationFormIfValid` function from the example above does three things at once:

- It handles the form submission event;
- Validates the data from the form;
- Creates and sends a new order.

Each step is probably important enough to be reflected in the name, but this bloats the name. It's better to think about how to split the task into smaller ones and separate the responsibility between the individual functions:

```js
// Serializes form data into an object:
function serializeForm() {}

// Validates the data object:
function validateFormData() {}

// Create a new order object from the data:
function createOrder() {}

// Sends the order to the server:
async function sendOrder() {}

// Handles the DOM event by calling other functions:
function handleOrderSubmit() {}
```

Then, instead of one big function trying to do _everything_, we would have a chain of different, smaller ones. Inner functions would contain actions _grouped by meaning_. Their names could take some of the details from the root function name, which would make it easier:

```js
async function handleOrderSubmit(event) {
  const formData = serializeForm(event.target);
  const validData = validateFormData(formData);
  const order = createOrder(validData);
  await sendOrder(order);
}
```

It seems like we're making the root function name less accurate because we're taking the details out of it. But it's all a matter of what kind of details we take away.

A good tactic is to see the function name from the _caller's_ perspective. Does the form care _how exactly_ its submission is handled? Probably not; it only matters that the submission is handled in principle. The handler might want to express _what form_ it's going to handle in its name:

```
handle          +
submit event    +
on order form   =
-----------------
handleOrderSubmit
```

...But _how exactly_ this happens is implementation details of `handleOrderSubmit`. These details aren't actually needed in the name because they are only important inside the function. And there, they can be expressed with inner function names.

| By the way 👀                                                                                                                                                 |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| You probably recognized this as an example of separating abstraction layers.[^abstractionlayers] We'll talk more about this in one of the following chapters. |

So basically when we see a long function name, we should figure out exactly what it's trying to tell us. We can try and take out all the details the name carries and divide them into 2 groups: “important outside” and “important inside.”

The details from the first group are better kept in the name. The details from the second group can be represented as inner function names. This helps to break down the task into simpler ones and group related functionality together.

| By the way 🛠                                                                                                                                     |
| :----------------------------------------------------------------------------------------------------------------------------------------------- |
| The logic of the function itself may still be complex. But there are usually fewer problems with naming if related details are grouped together. |

### Option for Naming Functions

When it comes to functions, the A/HC/LC pattern helps to maneuver between “too short” and “too long” names.[^ahclc] This pattern suggests combining the action, its subject, and its object:

```
prefix? + action (A) + high context (HC) + low context? (LC)
```

It can be used as a reference or starting point for naming functions. It's not always possible to follow the pattern so we can deviate from it and change the name if needed.

## Different Entities with Identical Names

During refactoring, we should also pay attention to the feeling of “confusion”. It can occur when different entities in the code have identical names.

When reading, we mostly rely on the names of classes, functions, and variables to understand the meaning of the code. If the name doesn't correlate with a variable 1 to 1, we have to figure out its meaning on the fly every time. To do this, we have to remember a lot of “meta-information” about variables. This makes code harder to read.

| By the way 🎫                                                                                                                                                                |
| :--------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| In some cases, the same applies to different names for the same variable. For example, it's better to avoid calling domain entities different names, but more on that later. |

To make the code easier to read, it's better to use _different_ names for _different_ entities:

```js
// Here, `user` is an object with user data:
function isOldEnough(user, minAge) {
  return user.age >= minAge;
}

// And here, `user` is a user name:
function findUser(user, users) {
  return users.find(({ name }) => name === user);
}

// At first glance, it's hard to tell them apart,
// because the identical names force us to think
// that the variables have the same meaning.
// This can be misleading.

// To avoid this, we have to always remember
// the difference between how each variable is used.

// It's better to express the meaning of the variable
// more precisely directly through its name:
function findUser(userName, users) {
  return users.find(({ name }) => name === userName);
}
```

Identical names for different entities are especially dangerous if they're located close to each other. The close location creates a “common context” for the variables, which only reinforces the feeling of sameness.

| However 🦦                                                                                                                                                                                                            |
| :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| There are exceptions to this rule, of course. For example, we can infer the difference from the type system or the context of use. Although, _by default_, it's better to use different names for different entities. |

## Ubiquitous Language

_Ubiquitous language_ can help us fight naming problems. It is a set of terms that describe the domain, which is used by the whole team. “The whole team” includes not only developers but the whole product team, including designers, product owners, stakeholders, etc.

| By the way 🎙                                                                                                                                                |
| :---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| The term itself comes from the _Domain-Driver Design, DDD_ methodology.[^ddd] I find it useful for describing business logic, you might find it useful too. |

In practice, ubiquitous language tells us that if “business people” use the term “Order” to describe orders, then we should use this word to describe orders in our code, tests, documentation, and verbal communication.

The power of ubiquitous language is _unambiguity_. If everyone calls a thing by the same name, less will be lost in the “translation between business and development.”

| Read more 📚                                                                                                                |
| :-------------------------------------------------------------------------------------------------------------------------- |
| More about this you can find in “Domain Modelling Made Functional” by Scott Wlaschin.[^dmmf] I highly recommend reading it. |

When we use terms close to reality, the mental model of a program becomes closer to _what_ it's modeling. In this case, it's much easier to spot incorrect program behavior.

## Lying Names

Sometimes during refactoring, we can find names that aren't accurate. Inaccuracy may vary from slight discrepancy to complete lies. Inaccurate names are better to be fixed as soon as possible.

If we're not sure how to properly call the variable, we should write down the reasons why the current name doesn't fit. For example, if we come across code like this:

```js
const trend = currentValue - previousValue;

// “Trend” here is the difference between
// “current” and “previous” values.
```

...And we've noticed in conversations that the team uses the term “Delta” (not “Trend”), then it's worth mentioning this inaccuracy. We might tell something like:

> - The term “Trend” probably doesn't accurately describe the value;
> - The team more often uses the term “Delta”, including product owners;
> - The project specifics may require trends built from more than two points.

We can raise these concerns to the team and consider renaming. If the name obviously lies, the discussion can be skipped. But when in doubt, it's worth discussing the concern with other developers and the product owner first.

## Domain Types

In languages with static typing, we can use types as a way to convey details about the domain. This takes the load off the entity name by putting some of the details into the type. The name becomes more succinct, but because of the type it hardly loses any meaning.

| By the way 💬                                                                                                                                                                                                  |
| :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| We could argue by saying that the type is only visible in signatures or in IDE popups, while the name is always visible.                                                                                       |
| Yes, but multiple longer names would make the code noisy, and their value would disappear. And secondly, I usually extract details that aren't needed _instantly_ and can be painlessly looked up in the type. |
| For me, it seems like a perfectly acceptable compromise.                                                                                                                                                       |

For example, it's convenient to use types to describe various data states that will appear in the application throughout its life cycle.

```ts
type CreatedOrder = {
  createdAt: TimeStamp;
  createdBy: UserId;
  products: ProductList;
};

type ProcessedOrder = {
  createdAt: TimeStamp;
  createdBy: UserId;
  products: ProductList;
  address: Delivery;
};
```

These types describe different data states (essentially different entities) with different names. But also, they disallow using the “wrong” type where it doesn't fit. For example, we can forbid attempts to send unprepared orders:

```ts
function sendOrder(order: ProcessedOrder) {
  // ...
}

const order: CreatedOrder = {
  /*...*/
};

// Function call below won't compile
// because the argument type doesn't satisfy the signature.
// It can be translated as: “The order isn't ready yet to be sent”.
sendOrder(order);
```

| Boolean flags? 🚩                                                                                                                                 |
| :------------------------------------------------------------------------------------------------------------------------------------------------ |
| We will discuss why using Boolean flags for denoting different states is less preferable than different types in the chapter about static typing. |

[^abstractionlayers]: Abstraction layer, Wikipedia, https://en.wikipedia.org/wiki/Abstraction_layer
[^ahclc]: Naming functions, A/HC/LC Pattern, https://github.com/kettanaito/naming-cheatsheet#ahclc-pattern
[^ddd]: “Domain-Driven Design” by Eric Evans, https://www.goodreads.com/book/show/179133.Domain_Driven_Design
[^dmmf]: “Domain Modeling Made Functional” by Scott Wlaschin, https://www.goodreads.com/book/show/34921689-domain-modeling-made-functional
