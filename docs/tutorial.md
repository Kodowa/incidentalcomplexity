## Purpose 

Hello everyone!

Today we are soliciting community feedback on our current syntax proposal. In this RFC I'll present the syntax, and provide a little rationale behind some of the decisions we've made. If you have the inclination, try writing some hypothetical code (we don't have a parser yet) and let us know how you find the experience (better yet, post the code and tell us what you intended it to do). Please also feel free to suggest any changes or additional syntax you might prefer. 

In issuing this RFC, we hope to specifically solicit comments on the following: 

1. How easy is it to read and understand code you haven't written in the proposed syntax? 
2. How easy is it to write code in the proposed syntax? 
3. Is the programming model clear?  
4. Does the syntax support working and thinking in this model? 
5. What changes would you make to writing or reading easier? 

## Intended Audience 

The first thing to note is this syntax is intended for developers. Its primary users will be us initially, but it needs to be accessible to people who are competent developers but don't know how Eve works. One of our primary design goals was to make it very easy to read, so we decided to use as little syntax as possible. In the same vein, white space has semantic meaning (like python but a little different). We also wanted the syntax to be familiar in a sense, but also very different from languages like Javascript, where you think with an imperative mindset. For all its simplicity, Eve does require a different mindset when programming, and trying to use features as you would in other languages would lead to unexpected results. Particularly, we are targeting developers of web and mobile applications, so our examples are biased toward that domain, but we hope that the syntax could be used beyond those bounds. 

## Eve 

Before we talk about the details of the syntax it's important to define what Eve actually is and how it works. In this context, it's fair to think of Eve as replacing as much of the web stack as possible. With Eve, you no longer need to configure a web server; you don't need to know HTML, CSS, or JavaScript; you don't need to install or maintain a database; you don't need to know any reactive JavaScript frameworks; etc. Eve handles all this with a single language. 

Functionally, Eve is a service that listens for messages in the form of queries. A query can either add facts to Eve, or ask about facts already in Eve. So then an Eve program is just a series of queries that transform the data stored in Eve. The syntax we propose was designed to express programs in this form.

### Facts

Facts are the basic building blocks of Eve. Each fact is an `(entity, attribute, value)` triple, or EAV for short. Entities can be thought of as structs or objects, each with any number of attributes, each with any number of values. For instance, Steve is 29 years old. This fact might be represented as the EAV `(Steve, age, 29)`. Steve also has two sisters, Sally and Sue, which in EAV form is `(Steve, sister, Sally)` and `(Steve, sister, Sue)`.   

### Sets

A set is a collection of elements, where each element is unique. For example, the tuple `(1, 2, 3)` is a set, but `(1, 1, 2, 3)` is not. Sets also have no ordering, so the set `{1, 2, 3}` (sets are usually denoted by curly braces) is the same as the set `{3, 2, 1}`. Eve follows set semantics, meaning collections of values in Eve are treated as sets. For instance, if Sally and Sue are both 27 years old, then if we ask Eve for the ages of Steve's sisters, we get back the set `{27}` with a single element. If we really wanted two elements, we would have to ask for something else in addition to the age to make each element unique. In this case, asking for the name and age of each sister would yield `{{Sally, 27}, {Sue, 27}}`.

### Variables

In Eve, variables work differently than in other languages.

## Proposed Syntax

### Hello world

An Eve program is a collection of queries. Each query typically asks eve to mutate data in some way. The typical pattern for a query is to select data from Eve, and then perform some mutation to the facts in Eve as a result. The "hello world" of Eve might look something like this:

```
add a phrase to Eve            // The query header can be anything
  add                          // Mutate the database by adding the following children
    #div text: "Hello world!"  // This div tag is added to the database with a text attribute
```

### Queries

Queries are written in a tree-like structure, where increased indention level indicates a parent-child relationship. Every query has a header, which is the root of the query and starts at indention-level 0. Query headers are the only lines that can be at indention level 0. Every other line in the query must be indented at least once. To be concise, we will omit the query header in examples to follwo.

### Selection

Selection in Eve is performed either by asking for individual entities, or asking for a group of entities that share similar attributes. This is accomplished using the name-selector `@`, or the tag-selector `#`, respectively.

Name Selector - The symbol `@` is used to select entities with a given name. For instance, if an entity were named "Steve", I would select it using `@Steve`. This creates a handle to the entity `Steve`, providing access to all its attributes. Attributes are selected by adding them as children to the selected entity: e.g.

```
@Steve
  age
  height
```
This statement would select `Steve`'s `age` and `height`. The following is also valid:

```
@Steve age height 
```
Where `age` and `height` are still children of `Steve`, even though they are on the same line. Selections can be composed as well. For example:

```
@Steve
  sister
    age
```
In this case `age` is a child of `sister`, so it would be the ages of all Steve's sisters. Be careful though, because in Eve, giving two things the same name means they are the same thing. For instance:

```
@Steve
  age
  sister
    age
```
Since `age` is a child of Steve and sister, this statement would select the sisters' ages that are the same as Steve's. In words: "Select the entity Steve, his age, and his sisters' ages, where each selected sister is the same age as Steve." If no sisters have the same age as Steve, then this statement selects nothing. An alias would prevent this from happening. e.g.
```
@Steve
  age
  sister
    age: sister-age
```
In this new query `sister-age` is a free variable that has been bound using the `:` operator. Binding also works with constants and math expressions
```
@Steve
  age
  sister
    age: age * 2
``` 

### Mutation

Mutation generally refers to changing the the facts stored in Eve. The available operations are one of add, update, or remove.

* Add - Store a new fact in Eve.  
* Update - Change a fact already stored in Eve
* Remove - Deletes a fact stored in Eve

## Proposed Syntax

Let's just dive in with an example of a program written in the proposed syntax, and I'll explain as we go along.

```
what year was sally born? 
  @Sally  
    year-born: year 
  add 
    #div 
      parent: "root" 
      text: "Sally was born in {year}." 
      
add some facts to Eve 
  add forever 
    #person  
      Name    year-born sibling 
      "Steve" 1986      @Sally 
      "Sally" 1988      @Steve 

find the year each person was born 
  #person year-born
  #time year
  add 
    person age: year – year-born 
```

Here we have three queries represented by three tree-like blocks of code. The first thing to note is that the whitespace has semantic meaning. Here, indention level indicates a parent-child relationship. 

Let’s go through these queries one by one and go into more detail 

```
what year was sally born?        // Query root, must be at 0 indention 
  @Sally                         // Name selector, handle to entity named "Sally" 
    age: sally-age               // Child year-born is attribute of sally. Bound to year 
  // Display a sentence          // Comment 
  add                            // Add children to database 
    #div                         // Div tag 
      parent: "root"             // Child attribute parent, value "root" 
      text: "Sally was born in {sally-age}." 
```

Query 1 asks Eve for the year Sally was born, and renders it as text to a webpage. Line 1 is the query root, and it is the only line that can be at indention level 0. The root line is treated as a comment, and can be a string of any text. This line is not a handle to the query or an identifier, so it does not have to be unique. We use this line to describe the intent of the query. 

The second line is indented once, indicating it is a child of the query root. This line uses the name selector, "@", followed by an identifier. In this case we get a handle to the entity named "Sally". Entities act a lot like structs or objects, in that they have members (called attributes) with values. To access attributes of entities, simply add them as a child. Here, on line 3 we access the year-born attribute of Sally. The : indicates we are binding it to another value, in this case year. Binding to free variables is a simple way of aliasing attributes. Attributes can even be other entities, and we can access attributes on them in the same way. E.g. 

```
@Sally 
  year-born 
  sibling 
    year-born: brother-year-born 
```

Note that if we did not alias sibling year-born, we would implicitly be binding sibling year-born to Sally year-born. This would yield only siblings the same age as her. 

Line 4 is an example of a comment. They work like you expect. 

Line 5 introduces a mutator. The possible mutators are "add", "update", and "remove", and they can only be children of the query root. The mutator is applied to each of its children. Here, a div is added with attributes parent and text. "#" is the tag selector. #div indicates all entities with the attribute "tag" = "div". You can apply multiple tags at once. For example, we could have said 

```
add 
  #div, #message, parent: "root", text: "Sally was born in {year}."
``` 

The children parent and text map to properties of the div tag. The parent property is bound to "root", which will cause this div to render at the DOM root. The text property uses the string interpolation operator "{}", which renders its contents as text. The cardinality (i.e. the number of elements in a set) of the variable in the operator is important to note. If this were our only query, the cardinality of Sally would be 0, because there would be no Sally in the system. If we look ahead to Query 2, we can see how Sally gets into the system:

```
add some facts to Eve 
  add forever                     // Make the fact persist in the Evedb 
    #person                       // Tag entities with person 
      name    year-born sibling   // Attributes are name, age, sibling 
      "Steve" 1986      @Sally    // Add steve and his attributes. 
      "Sally" 1988      @Steve    // Add Sally. @Steve references the entity steve
```

Query 2 adds some facts to Eve: two entities tagged "person", one named Steve and the other Sally, who are siblings and age 30 and 27 respectively. The optional keyword "forever" is added after a mutator to indicate that the mutation should persist beyond the lifetime of the query. This shorthand allows us to add each entity in a tabular way. We could have also done the following:

```
add some facts to Eve 
  add forever                 
    #person name: "Steve" year-born: 1986 siblings: @Sally
    #person name: "Sally" year-born: 1988 siblings: @Steve
```
Which is also completely valid. This format is intended for the case where the mutations are not uniform. You might notice that the Query 1 asks for Sally's year-born, while we only added her age to Eve. That's okay, because if we know today's date, we can derive year-born, which is what we do in Query 3.  

```  
find the year each person was born 
  #person year-born               // Select name, age pairs 
  #time year                      // Select the time tag, whose attributes represent the current time 
  add                             // Add a transient fact 
    person age: year – year-born  // Add the calculated attribute age to every person with a year-born
```
Query 3 calculates the year each person was born. This query says that for every name, age pair of entities tagged person (line 1), add the attribute year-born, which is equal to the current year minus the person’s age (lines 4-5). So even though we did not add year-born to Eve in Query 2, it is available for any query to ask about (Query 1 for instance).

Also, pay special attention to line 3, where we select #time, which is a built-in tag that listens to the system clock and updates its attributes to match. This is an example of how Eve interacts with the outside world. In general.

## Advanced Syntax

The three queries examined so far represent a complete Eve program; Query 1 asks fors data and displays it, Query 2 adds facts to Eve, and Query 3 derives implications from base facts. In general, most Eve programs can be built using these three basic patterns. In this section, we introduce a few more concepts

### Aggregates

### Choose

### Union

### Not

## Grammar Overview 

Here we present the EBNF grammar of the Eve developer syntax

```
program = {query}; 
query = header, {line}; 
header = character, {word}, '\n';
line = ('\t', {ws}, expression | '\t', mutation), '\n';
expression = binding | name-expression | tag-expression;

binding = identifier, [(':', (math-expression | filter-expression))];
tag-expression = tag, {(ws, (tag | binding))};
name-expression = name, {(ws, (tag | binding))}; 
math-expression = math-expression, ' ', operator, ' ', math-expresison | '(', math-expression, ')' | literal | identifier | function;
filter-expression = math-expression, filter, math-expression;
function = identifier, '(', {(literal | identifier), {ws} ,')';
mutation = mutator, '\n', '\t', '\t', {(tag-expression | name-expression)}

tag = '#', identifier | identifier;
name = '@', identifier | identifier;  

keyword = 'not' | 'choose' | 'or' | 'union' | 'and' | 'given' | 'per' | mutator; 
mutator = ('add' | 'update' | 'remove'), [(' ', 'forever')];
identifier = {character} - keyword;
 
operator = '+' | '-' | '*' | '/' | '^'
filter = '<' | '<=' | '=' | '!=' | '>=' | '>' 
nl = '\n';
ws = '\n' | '\t' | ' ' | ';' | ','; 
character = ? any visible character ?;
digit = ? any digit ?; 
word = {character}, [' '];
number = digit, {digit}, [('.', digit, {digit})];
string = '"',{(character | ' ')},'"';
literal = (number | string)
```