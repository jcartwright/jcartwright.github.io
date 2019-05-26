{% highlight elixir %}
## lib/comps_api_web/controllers/stripe/event_controller.ex

defmodule CompsAPIWeb.StripeEventController do
  use CompsAPIWeb, :controller

  alias CompsAPI.Payments.Events

  def create(conn, _params) do
    with %Stripe.Event{} = event <- conn.assigns.event do
      Task.Supervisor.async_nolink(CompsAPIWeb.TaskSupervisor, fn ->
        Events.handle_event(event)
      end)

      conn
      |> send_resp(:ok, "")
    end
  end
end
{% endhighlight %}
