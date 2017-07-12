## Don't reference `Repo` in a controller

### Don't do this:

```elixir
def show(conn, %{"id" => id}) do
  post = MyApp.Repo.get(Post, id)
  render(conn, "show.html", post: post)
end
```

### Instead, do this:

```elixir
def show(conn, %{"id" => id}) do
  post = MyApp.Blog.get_post(id)
  render(conn, "show.html", post: post)
end
```

Ecto's `Repo` is an implementation detail internal to your application that should not be exposed to the web layer. Data persistence can be backed by many things: a relational database, a document database, an ETS table, a flat file, an external API, a GenServer… the list goes on. Leave it up to the Context to hide that detail from your external web interface.

## Make the first function argument represent the data being acted ON

### Don't do this:

```elixir
MyApp.Posts.update_post(params, post)
```

### Instead, do this:

```elixir
MyApp.Blog.update_post(post, params) # Or, if you prefer: post |> MyApp.Posts.update(params)
```

Elixir isn't object-oriented, but the pipe operator `|>` lets us mutate data in a functional way that still feels like calling a method on an object. Take advantage of that exepectation by designing your functions to take the data to be mutated (or a pointer to that data, like an `id`) as the first parameter.
