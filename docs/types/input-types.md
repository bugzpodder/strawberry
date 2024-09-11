---
title: Input types
---

# Input types

In addition to [object types](./object-types) GraphQL also supports input types.
While being similar to object types, they are better suited for input data as
they limit the kind of types you can use for fields.

This is how the
[GraphQL spec defines the difference between object types and input types](https://spec.graphql.org/June2018/#sec-Input-Objects):

> The GraphQL Object type (ObjectTypeDefinition)... is inappropriate for re‐use
> (as input), because Object types can contain fields that define arguments or
> contain references to interfaces and unions, neither of which is appropriate
> for use as an input argument. For this reason, input objects have a separate
> type in the system.

## Defining input types

In Strawberry, you can define input types by using the `@strawberry.input`
decorator, like this:

<CodeGrid>

```python
import strawberry


@strawberry.input
class Point2D:
    x: float
    y: float
```

```graphql
input Point2D {
  x: Float!
  y: Float!
}
```

</CodeGrid>

Then you can use input types as argument for your fields or mutations:

```python
import strawberry


@strawberry.type
class Mutation:
    @strawberry.mutation
    def store_point(self, a: Point2D) -> bool:
        return True
```

If you want to include optional arguments, you need to provide them with a
default. For example if we want to expand on the above example to allow optional
labeling of our point we could do:

<CodeGrid>

```python
import strawberry
from typing import Optional


@strawberry.input
class Point2D:
    x: float
    y: float
    label: Optional[str] = None
```

```graphql
type Point2D {
    x: Float!
    y: Float!
    label: String = null
}
```

</CodeGrid>

Alternatively you can also use `strawberry.UNSET` instead of the `None` default
value, which will make the field optional in the schema:

<CodeGrid>

```python
import strawberry
from typing import Optional


@strawberry.input
class Point2D:
    x: float
    y: float
    label: Optional[str] = strawberry.UNSET
```

```graphql
type Point2D {
  x: Float!
  y: Float!
  label: String
}
```

</CodeGrid>

## API

`@strawberry.input(name: str = None, description: str = None)`

Creates an input type from a class definition.

- `name`: if set this will be the GraphQL name, otherwise the GraphQL will be
  generated by camel-casing the name of the class.
- `description`: this is the GraphQL description that will be returned when
  introspecting the schema or when navigating the schema using GraphiQL.

## One Of Input Types

Strawberry also supports defining input types that can have only one field set.
This is based on the
[OneOf Input Objects RFC](https://github.com/graphql/graphql-spec/pull/825)

To define a one of input type you can use the `one_of` flag on the
`@strawberry.input` decorator:

<CodeGrid>

```python
import strawberry


@strawberry.input(one_of=True)
class SearchBy:
    name: str | None = strawberry.UNSET
    email: str | None = strawberry.UNSET
```

```graphql
input SearchBy @oneOf {
  name: String
  email: String
}
```

</CodeGrid>

## Deprecating fields

Fields can be deprecated using the argument `deprecation_reason`.

<Note>

This does not prevent the field from being used, it's only for documentation.
See:
[GraphQL field deprecation](https://spec.graphql.org/June2018/#sec-Field-Deprecation).

</Note>

<CodeGrid>

```python
import strawberry
from typing import Optional


@strawberry.input
class Point2D:
    x: float
    y: float
    z: Optional[float] = strawberry.field(
        deprecation_reason="3D coordinates are deprecated"
    )
    label: Optional[str] = strawberry.UNSET
```

```graphql
input Point2D {
  x: Float!
  y: Float!
  z: Float @deprecated(reason: "3D coordinates are deprecated")
  label: String
}
```

</CodeGrid>