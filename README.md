## Don't reference `Repo` from a controller, view, or channel

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
MyApp.Blog.update_post(params, post)
```

### Instead, do this:

```elixir
MyApp.Blog.update_post(post, params) # Or, if you prefer: post |> MyApp.Posts.update(params)
```

Elixir isn't object-oriented, but the pipe operator `|>` lets us mutate data in a functional way that still feels like calling a method on an object. Take advantage of that exepectation by designing your functions to take the data to be mutated (or a pointer to that data, like an `id`) as the first parameter.

## Define queries directly on the schema module

```elixir
# lib/my_app/blog/post.ex
defmodule MyApp.Blog.Post do
  import Ecto.Query
  
  def published(query \\ Post) do
    from p in query, where: not is_nil(p.published_at)
  end
  
  def ordered(query \\ Post) do
    from p in query, order_by: p.published_at
  end
end

# lib/my_app/blog/blog.ex
defmodule MyApp.Blog do
  def list_posts do
    Post
    |> Post.published
    |> Post.ordered
    |> Repo.all
  end
end
```

Defining simple, composable queries directly on the schema module is very convenient for building up more complex queries at higher levels.

## Pass the current user directly into authenticated controller actions

```elixir
# lib/my_app/web.ex
defmodule MyApp.Web do
  # ...Phoenix-generated scopes here...
  
  def authenticated_controller do
    quote do
      use MyApp.Web, :controller
      
      # Example using Guardian
      plug Guardian.Plug.EnsureAuthenticated, handler: MyApp.Web.AuthHandler
  
      # Override Phoenix.Controller.action/2 callback
      def action(conn, params) do
        args = [conn, params, Guardian.Plug.current_resource(conn)]
        apply(__MODULE__, action_name(conn), args)
      end
    end
  end
end

# lib/my_app/web/controllers/post_controller.ex
defmodule MyApp.Web.PostController do
  # Use the new scope
  use MyApp.Web, :authenticated_controller
  
  # Notice the extra parameter
  def new(conn, params, current_user) do
    changeset = MyApp.Blog.new_post(current_user, params)
    render(conn, "new.html", changeset: changeset)
  end
end
```

By overriding the default Phoenix controller `action/2` callback, you can inject custom parameters into each controller action. For authenticated controllers, knowing the `current_user` is often required, so pass it in directly to the function for convenience.
