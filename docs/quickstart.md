# Eve Quickstart Tutorial

This tutorial is intended to get you started with Eve as quickly as possible. So, let's get started!

## Intro to Eve

Before you begin using Eve, it is important to understand how it is different from other programming languages you've used. The foundation of Eve is [relational algebra](https://en.wikipedia.org/wiki/Relational_algebra), which means that Eve programs are written by specifying data and the relationships between them. Don't worry, as a programmer, you needn't write any symbolic algebra at all; the behavior of a program is determined by data alone. This sets Eve apart from imperative languages like C or Java, where control statements like loops and subroutines define program behavior (for an introduction to the difference between Eve and imperative languages, see this [tutorial](http://localhost)).

## Eve Programs

Think of Eve as an agent helping you with your program. Eve responds to only two commands:

1. What facts do you know?
2. Remember a new fact.

That's it, just those two things. The first case is called querying, where you ask a question and get Eve gives you answer. The second case is mutation, where you can add new facts to Eve, which includes updating previously known facts, and removing facts from Eve entirely.

## Querying

Queries ask about what facts Eve knows. Eve stores data is stored as `(Entity, Attribute, Value)` tuples, also called EAVs.

### Entity Selector ( `@` ) 

The entitiy selector is used to select entities with a specific name. The statement `@Henrietta` selects the entity with the name "Henrietta". 

### Tag Selector ( `#` )

The tag selector is used to select all entities with a given tag. For example, the statement `#people` would select every entity with the tag "people". Tag selectors can be chained, meaning we can get the intersection of sets. The statement `#husband #father` would select all the entities that are tagged husband as well as father.

### Attributes

### Binding

## Mutation

### Add

### Remove

### Update