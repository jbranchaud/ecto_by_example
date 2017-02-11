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
> Repo.one(from d in "developers", select: fragment("count(*)"))

17:19:01.195 [debug] QUERY OK source="developers" db=1.0ms queue=2.9ms
SELECT count(*) FROM "developers" AS d0 []
32
```

> How many developers and posts are there?

```elixir
> post_count = from p in "posts", select: %{count: count(p.id)}
#Ecto.Query<from p in "posts", select: %{count: count(p.id)}>

> developer_count = from d in "developers", select: %{count: count(d.id)}
#Ecto.Query<from d in "developers", select: %{count: count(d.id)}>

> Repo.one(from p in subquery(post_count), join: d in subquery(develope
r_count), select: %{post_count: p.count, developer_count: d.count})

17:39:26.373 [debug] QUERY OK db=12.4ms
SELECT s0."count", s1."count" FROM (SELECT count(p0."id") AS "count" FROM "posts" AS p0) AS s0 INNER JOIN (SELECT count(d0."id") AS "count" FROM "developers" AS d0) AS s1 ON TRUE []
%{developer_count: 32, post_count: 1066}
```
