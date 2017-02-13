# hr-til

### How many posts are there?

```elixir
> Repo.one(from p in "posts", select: count(p.id))
1066

17:16:36.573 [debug] QUERY OK source="posts" db=10.8ms queue=0.2ms
SELECT count(p0."id") FROM "posts" AS p0 []
```

### How many developers are there?

```elixir
> Repo.one(from d in "developers", select: fragment("count(*)"))

17:19:01.195 [debug] QUERY OK source="developers" db=1.0ms queue=2.9ms
SELECT count(*) FROM "developers" AS d0 []
32
```

### How many developers and posts are there?

```elixir
> post_count = from p in "posts", select: %{count: count(p.id)}
#Ecto.Query<from p in "posts", select: %{count: count(p.id)}>

> developer_count = from d in "developers", select: %{count: count(d.id)}
#Ecto.Query<from d in "developers", select: %{count: count(d.id)}>

> Repo.one(from p in subquery(post_count),
            join: d in subquery(developer_count),
            select: %{post_count: p.count, developer_count: d.count})

17:39:26.373 [debug] QUERY OK db=12.4ms
SELECT s0."count", s1."count" FROM (SELECT count(p0."id") AS "count" FROM "posts" AS p0) AS s0 INNER JOIN (SELECT count(d0."id") AS "count" FROM "developers" AS d0) AS s1 ON TRUE []
%{developer_count: 32, post_count: 1066}
```

### How many posts on average per developer?

First, let's write a query that groups are posts by `developer_id` and
counts the number of posts for each.

```elixir
> post_counts = from(p in "posts",
                 group_by: p.developer_id,
                 select: %{post_count: count(p.id), developer_id: p.developer_id})
#Ecto.Query<from p in "posts", group_by: [p.developer_id],
 select: %{post_count: count(p.id), developer_id: p.developer_id}>
```

We can see the result of this query with `Repo.all`.

```elixir
> Repo.all(post_counts)

10:29:09.177 [debug] QUERY OK source="posts" db=5.8ms
SELECT count(p0."id"), p0."developer_id" FROM "posts" AS p0 GROUP BY p0."developer_id" []
[%{developer_id: 14, post_count: 6}, %{developer_id: 25, post_count: 43},
 %{developer_id: 32, post_count: 1}, %{developer_id: 27, post_count: 2},
 %{developer_id: 8, post_count: 332}, %{developer_id: 17, post_count: 1},
 %{developer_id: 15, post_count: 23}, %{developer_id: 1, post_count: 1},
 %{developer_id: 10, post_count: 18}, %{developer_id: 26, post_count: 78},
 %{developer_id: 11, post_count: 15}, %{developer_id: 4, post_count: 130},
 %{developer_id: 18, post_count: 14}, %{developer_id: 30, post_count: 10},
 %{developer_id: 16, post_count: 3}, %{developer_id: 33, post_count: 1},
 %{developer_id: 6, post_count: 3}, %{developer_id: 19, post_count: 9},
 %{developer_id: 29, post_count: 82}, %{developer_id: 2, post_count: 236},
 %{developer_id: 23, post_count: 10}, %{developer_id: 31, post_count: 5},
 %{developer_id: 20, post_count: 8}, %{developer_id: 5, post_count: 3},
 %{developer_id: 13, post_count: 3}, %{developer_id: 22, post_count: 12},
 %{developer_id: 9, post_count: 10}, %{developer_id: 24, post_count: 4},
 %{developer_id: 7, post_count: 3}]
```

The `Repo.aggregate` function with the `:avg` atom will help us get our
average. By subquerying our query, the `post_count` alias is exposed as the
column that we can average on.

```elixir
> Repo.aggregate(subquery(post_counts), :avg, :post_count)

10:29:45.425 [debug] QUERY OK db=13.0ms queue=0.1ms
SELECT avg(s0."post_count") FROM (SELECT count(p0."id") AS "post_count", p0."developer_id" AS "developer_id" FROM "posts" AS p0 GROUP BY p0."developer_id") AS s0 []
#Decimal<36.7586206896551724>
```

### How many posts on average per developer (unnecessarily using joins)?

First, we need a query that will give us the post count for each developer.

```elixir
> post_counts = from(p in "posts", join: d in "developers", on: p.developer_id == d.id, group_by: d.id, select: %{post_count: count(p.id)})
#Ecto.Query<from p in "posts", join: d in "developers",
 on: p.developer_id == d.id, group_by: [d.id],
 select: %{post_count: count(p.id)}>
```

We can see the kind of result this query produces with `Repo.all`.

```elixir
> Repo.all(post_counts)

18:53:48.591 [debug] QUERY OK source="posts" db=12.8ms queue=1.6ms
SELECT count(p0."id") FROM "posts" AS p0 INNER JOIN "developers" AS d1 ON p0."developer_id" = d1."id" GROUP BY d1."id" []
[%{post_count: 6}, %{post_count: 43}, %{post_count: 1}, %{post_count: 2},
 %{post_count: 332}, %{post_count: 1}, %{post_count: 23}, %{post_count: 1},
 %{post_count: 18}, %{post_count: 78}, %{post_count: 15}, %{post_count: 130},
 %{post_count: 14}, %{post_count: 10}, %{post_count: 3}, %{post_count: 1},
 %{post_count: 3}, %{post_count: 9}, %{post_count: 82}, %{post_count: 236},
 %{post_count: 10}, %{post_count: 5}, %{post_count: 8}, %{post_count: 3},
 %{post_count: 3}, %{post_count: 12}, %{post_count: 10}, %{post_count: 4},
 %{post_count: 3}]
```

The `Repo.aggregate` function with the `:avg` atom will help us get our
average. By subquerying our query, the `post_count` alias is exposed as the
column that we can average on.

```elixir
> Repo.aggregate(subquery(post_counts), :avg, :post_count)

18:53:57.912 [debug] QUERY OK db=14.5ms
SELECT avg(s0."post_count") FROM (SELECT count(p0."id") AS "post_count" FROM "posts" AS p0 INNER JOIN "developers" AS d1 ON p0."developer_id" = d1."id" GROUP BY d1."id") AS s0 []
#Decimal<36.7586206896551724>
```

### Scrubbing Data

We are working with a production database, so it would be nice to scrub any
personal or sensitive data (e.g. emails and names).

First, let's build up a query that declares what fields we want to update
and how we want to update them.

```elixir
> dev_updates = from(d in devs, update: [
    set: [
               email: fragment("'developer' || ? || '@example.com'", d.id),
            username: fragment("'developer' || ?", d.id),
      twitter_handle: fragment("'at_developer' || ?", d.id),
          slack_name: fragment("'slack_developer' || ?", d.id)
    ]
  ])
#Ecto.Query<from d in "developers",
 update: [set: [email: fragment("'developer' || ? || '@example.com'", d.id), username: fragment("'developer' || ?", d.id), twitter_handle: fragment("'at_developer' || ?", d.id), slack_name: fragment("'slack_developer' || ?", d.id)]]>
```

Then we can evaluate this query with the `Repo.update_all` function. Using
the `:returning` option, we can see some of the updated results.

```elixir
> Repo.update_all(dev_updates, [], returning: [:email, :username])

16:29:08.220 [debug] QUERY OK source="developers" db=6.0ms
UPDATE "developers" AS d0 SET "email" = 'developer' || d0."id" || '@example.com', "username" = 'developer' || d0."id", "twitter_handle" = 'at_developer' || d0."id", "slack_name" = 'slack_developer' || d0."id" RETURNING d0."email", d0."username" []
{32,
 [%{email: "developer6@example.com", username: "developer6"},
  %{email: "developer4@example.com", username: "developer4"},
  %{email: "developer3@example.com", username: "developer3"},
  %{email: "developer10@example.com", username: "developer10"},
  %{email: "developer12@example.com", username: "developer12"},
  %{email: "developer14@example.com", username: "developer14"},
  ...]}
```

The result is a tuple. The first part tells us 32 records were updated. The
second part gives us a list of the update fields specified in the
`:returning` clause.

### Who are the top 10 posters, in terms of quantity of posts?

We can start by creating a partial query joining developers to posts. We'll
call it `developers_and_posts`.

```elixir
> developers_and_posts = from(d in "developers", join: p in "posts", on: p.developer_id == d.id)
#Ecto.Query<from d in "developers", join: p in "posts",
 on: p.developer_id == d.id>
```

Using `developers_and_posts` we can group by developer's `id`, selecting for
the `username` and count of posts for each developer.

```elixir
> Repo.all(from([d, p] in developers_and_posts, group_by: d.id, select: {d.username, count(p.id)}))

17:03:27.265 [debug] QUERY OK source="developers" db=8.4ms
SELECT d0."username", count(p1."id") FROM "developers" AS d0 INNER JOIN "posts" AS p1 ON p1."developer_id" = d0."id" GROUP BY d0."id" []
[{"developer14", 6}, {"developer25", 43}, {"developer32", 1},
 {"developer27", 2}, {"developer8", 332}, {"developer17", 1},
 {"developer15", 23}, {"developer1", 1}, {"developer10", 18},
 {"developer26", 78}, {"developer11", 15}, {"developer4", 130},
 {"developer18", 14}, {"developer30", 10}, {"developer16", 3},
 {"developer33", 1}, {"developer6", 3}, {"developer19", 9}, {"developer29", 82},
 {"developer2", 236}, {"developer23", 10}, {"developer31", 5},
 {"developer20", 8}, {"developer5", 3}, {"developer13", 3}, {"developer22", 12},
 {"developer9", 10}, {"developer24", 4}, {"developer7", 3}]
```

This leaves us with a jumble of results. We want the top 10 posters which
means we need to order the results by post count and then truncate the list
at 10.

We can achieve the ordering with an `order_by` clause. Limiting our results
can be done with a `limit` clause.

```elixir
> Repo.all(from([d, p] in developers_and_posts,
            group_by: d.id,
            order_by: [desc: count(p.id)],
            limit: 10,
            select: {d.username, count(p.id)}))

17:17:51.752 [debug] QUERY OK source="developers" db=5.2ms
SELECT d0."username", count(p1."id") FROM "developers" AS d0 INNER JOIN "posts" AS p1 ON p1."developer_id" = d0."id" GROUP BY d0."id" ORDER BY count(p1."id") DESC LIMIT 10 []
[{"developer8", 332}, {"developer2", 236}, {"developer4", 130},
 {"developer29", 82}, {"developer26", 78}, {"developer25", 43},
 {"developer15", 23}, {"developer10", 18}, {"developer11", 15},
 {"developer18", 14}]
```

### What are the top 10 most liked Elixir posts?

We can create a partial query, `top_posts`, that represents the most liked
posts.

```elixir
> top_posts = from(p in "posts", order_by: [desc: p.likes])
#Ecto.Query<from p in "posts", order_by: [desc: p.likes]>
```

We can then build on this by joining in the `channels` table to filter by
posts posted to the Elixir channel. This is accomplished with a `where`
clause on channel `name`. We'll call this `top_elixir_posts`.

```elixir
> top_elixir_posts = from(p in top_posts,
                      join: c in "channels",
                      on: p.channel_id == c.id,
                      where: c.name == "Elixir")
#Ecto.Query<from p in "posts", join: c in "channels", on: p.channel_id == c.id,
 where: c.name == "Elixir", order_by: [desc: p.likes]>
```

Now that we've filtered on the channel name, we don't have to think about
the `channels` table or a binding to it anymore. We'll just limit the
results to 10 and select the columns we are interested in.

```elixir
> Repo.all(from(p in top_elixir_posts, limit: 10, select: {p.likes, p.title}))

17:58:50.679 [debug] QUERY OK source="posts" db=2.7ms
SELECT p0."likes", p0."title" FROM "posts" AS p0 INNER JOIN "channels" AS c1 ON p0."channel_id" = c1."id" WHERE (c1."name" = 'elixir') ORDER BY p0."likes" DESC LIMIT 10 []
[{15, "Named Captures with Elixir Regular Expressions"},
 {14, "Create A Date With The Date Sigil"},
 {14, "Requiring Keys For Structs"},
 {14, "Execute Raw SQL In An Ecto Migration"},
 {14, "Invoke Elixir Functions with Apply"},
 {12, "Weird Operator Uses in Elixir"},
 {12, "List Functions For A Module"},
 {12, "Three data types that go `into` a Map"},
 {12, "Pry in Elixir Phoenix"},
 {12, "Capture IO.puts in ExUnit tests"}]
```

### What is the oldest (first) Elixir post?

We can start with a join between the `posts` and `channels` tables in order
to filter by posts in the Elixir channel. We'll call this partial query
`elixir_posts`.

```elixir
> elixir_posts = from(p in "posts", join: c in "channels", on: c.id == p.channel_id, where: c.name == "elixir")
#Ecto.Query<from p in "posts", join: c in "channels", on: c.id == p.channel_id,
 where: c.name == "elixir">
```

We then order our `elixir_posts` by the date they were created, select the
fields we are interested in, and limit the result set to 1.

```elixir
> Repo.all(from(p in elixir_posts, order_by: [asc: p.created_at], select: {p.created_at, p.title}, limit: 1))

18:05:45.846 [debug] QUERY OK source="posts" db=4.4ms
SELECT p0."created_at", p0."title" FROM "posts" AS p0 INNER JOIN "channels" AS c1 ON c1."id" = p0."channel_id" WHERE (c1."name" = 'elixir') ORDER BY p0."created_at" LIMIT 1 []
[{{{2015, 10, 9}, {13, 57, 4, 20409}}, "Pry in Elixir Phoenix"}]
```
