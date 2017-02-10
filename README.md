# ecto_by_example

## Migrations

> Create a table with some basic Ecto data types

```elixir
create table(:pokemons) do
  add :name, :string, null: false
  add :level, :integer, null: false, default: 1
  add :special, :float, null: false, default: 1.0
  add :wild, :boolean, null: false, default: true
end
```

```sql
                                 Table "public.pokemons"
 Column  |          Type          |                       Modifiers
---------+------------------------+-------------------------------------------------------
 id      | integer                | not null default nextval('pokemons_id_seq'::regclass)
 name    | character varying(255) | not null
 level   | integer                | not null default 1
 special | double precision       | not null default 1.0
 wild    | boolean                | not null default true
Indexes:
    "pokemons_pkey" PRIMARY KEY, btree (id)
```

> Create a table with some database-specific data types

```elixir
create table(:pokemons) do
  add :name, :varchar, null: false
  add :level, :smallint, null: false, default: 1
  add :special, :decimal, null: false, default: 1.0
  add :wild, :boolean, null: false, default: true
end
```

```sql
                               Table "public.pokemons"
 Column  |       Type        |                       Modifiers
---------+-------------------+-------------------------------------------------------
 id      | integer           | not null default nextval('pokemons_id_seq'::regclass)
 name    | character varying | not null
 level   | smallint          | not null default 1
 special | numeric           | not null default 1.0
 wild    | boolean           | not null default true
Indexes:
    "pokemons_pkey" PRIMARY KEY, btree (id)
```

> Create a table with timestamps generated by Ecto

```elixir
create table(:trainers) do
  add :name, :varchar, null: false

  timestamps()
end
```

```sql
                                      Table "public.trainers"
   Column    |            Type             |                       Modifiers
-------------+-----------------------------+-------------------------------------------------------
 id          | integer                     | not null default nextval('trainers_id_seq'::regclass)
 name        | character varying           | not null
 inserted_at | timestamp without time zone | not null
 updated_at  | timestamp without time zone | not null
Indexes:
    "trainers_pkey" PRIMARY KEY, btree (id)
```

> Create a table with custom timestamps

```elixir
create table(:trainers) do
  add :name, :varchar, null: false
  add :created_at, :timestamptz, null: false
  add :updated_at, :timestamptz, null: false
end
```

```sql
                                    Table "public.trainers"
   Column   |           Type           |                       Modifiers
------------+--------------------------+-------------------------------------------------------
 id         | integer                  | not null default nextval('trainers_id_seq'::regclass)
 name       | character varying        | not null
 created_at | timestamp with time zone | not null
 updated_at | timestamp with time zone | not null
Indexes:
    "trainers_pkey" PRIMARY KEY, btree (id)
```
