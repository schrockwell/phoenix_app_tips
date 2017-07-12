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
  post = MyApp.Posts.get(id)
  render(conn, "show.html", post: post)
end
```

Ecto's `Repo` is an implementation detail internal to your application that should not be exposed to the web layer. Data persistence can be backed by many things: a relational database, a document database, an ETS table, a flat file, an external API, a GenServer… the list goes on. Leave it up to the Context to hide that detail from your external web interface.
