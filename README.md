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
