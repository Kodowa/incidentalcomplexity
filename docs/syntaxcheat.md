# Eve Syntax Cheatsheet

## Structure

```
// - Begin a comment line
/* - Begin a comment block
*/ - End a comment block
\t - Parent -> Child relationship 
,  - Treated as whitespace
;  - Treated as whitespace
```

## Querying

```
@ - Entity Selector
# - Tag Selector
: - Binding
```

## Mutation

```
add       - Add children to Eve
remove    - Remove children from Eve
update    - Update a fact already in Eve to a new value
* forever - Use any mutator with the foever keyword to persist it in the database
``` 

## Keywords

```
choose - evaluates the first branch that returns results
or     - indicates a choose branch
union  - combines all branches into a single result
and    - indicates a untion branch
not    - Exclude children from results
```

## Strings

```
"" - encapsulates a string
{} - string interpolation
```