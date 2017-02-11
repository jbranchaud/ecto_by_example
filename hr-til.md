# hr-til

> How many posts are there?

```elixir
> Repo.one(from p in "posts", select: count(p.id))
1066

17:16:36.573 [debug] QUERY OK source="posts" db=10.8ms queue=0.2ms
SELECT count(p0."id") FROM "posts" AS p0 []
```

> How many developers are there?

```elixir
> Repo.one(from d in "developers", select: count(d.id))

17:19:01.195 [debug] QUERY OK source="developers" db=1.0ms queue=2.9ms
SELECT count(d0."id") FROM "developers" AS d0 []
32
```
