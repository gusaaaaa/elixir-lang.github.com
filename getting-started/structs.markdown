---
layout: getting-started
title: Structs
redirect_from: /getting-started/struct.html
---

# {{ page.title }}

{% include toc.html %}

In [chapter 7](/getting-started/keywords-and-maps.html) we learned about maps:

```iex
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

Structs are extensions built on top of maps that provide compile-time checks and default values.

## Defining structs

To define a struct, the `defstruct` construct is used:

```iex
iex> defmodule User do
...>   defstruct name: "John", age: 27
...> end
```

The keyword list used with `defstruct` defines what fields the struct will have along with their default values.

Structs take the name of the module they're defined in. In the example above, we defined a struct named `User`.

We can now create `User` structs by using a syntax similar to the one used to create maps:

```iex
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

Structs provide *compile-time* guarantees that only the fields (and *all* of them) defined through `defstruct` will be allowed to exist in a struct:

```iex
iex> %User{oops: :field}
** (CompileError) iex:3: unknown key :oops for struct User
```

## Accessing and updating structs

When we discussed maps, we showed how we can access and update the fields of a map. The same techniques (and the same syntax) apply to structs as well:

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "Meg"}
iex> %{meg | oops: :field}
** (KeyError) key :oops not found in: %User{age: 27, name: "Meg"}
```

When using the update syntax (`|`), the <abbr title="Virtual Machine">VM</abbr> is aware that no new keys will be added to the struct, allowing the maps underneath to share their structure in memory. In the example above, both `john` and `meg` share the same key structure in memory.

Structs can also be used in pattern matching, both for matching on the value of specific keys as well as for ensuring that the matching value is a struct of the same type as the matched value.

```iex
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

## Structs are bare maps underneath

In the example above, pattern matching works because underneath structs are bare maps with a fixed set of fields. As maps, structs store a "special" field named `__struct__` that holds the name of the struct:

```iex
iex> is_map(john)
true
iex> john.__struct__
User
```

Notice that we referred to structs as **bare** maps because none of the protocols implemented for maps are available for structs. For example, you can neither enumerate nor access a struct:

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john[:name]
** (UndefinedFunctionError) undefined function: User.fetch/2
iex> Enum.each john, fn({field, value}) -> IO.puts(value) end
** (Protocol.UndefinedError) protocol Enumerable not implemented for %User{age: 27, name: "John"}
```

However, since structs are just maps, they work with the functions from the `Map` module:

```iex
iex> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}
iex> Map.merge(kurt, %User{name: "Takashi"})
%User{age: 27, name: "Takashi"}
iex> Map.keys(john)
[:__struct__, :age, :name]
```

Structs alongside protocols provide one of the most important features for Elixir developers: data polymorphism. That's what we will explore in the next chapter.
