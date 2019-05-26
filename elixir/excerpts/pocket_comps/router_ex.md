{% highlight elixir %}
## lib/comps_api_web/router.ex

defmodule CompsAPIWeb.Router do
  use CompsAPIWeb, :router
  use Plug.ErrorHandler
  use Sentry.Plug

  pipeline :api do
    plug :accepts, ["json"]
  end

  pipeline :webhook do
    plug :accepts, ["json"]
  end

  # ...

  scope "/api/stripe", CompsAPIWeb do
    pipe_through :webhook

    post "/event", StripeEventController, :create
  end

  # ...

end
{% endhighlight %}
