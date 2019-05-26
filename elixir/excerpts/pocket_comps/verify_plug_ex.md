{% highlight elixir %}
## lib/comps_api_web/plug/verify_stripe_webhook.ex

defmodule CompsAPIWeb.Plug.VerifyStripeWebhook do
  use Plug.Builder

  require Logger

  import CompsAPIWeb.Router.Helpers

  @secret Application.get_env(:comps_api, __MODULE__)[:secret]

  def init(opts), do: opts

  def call(%{request_path: request_path} = conn, _opts) do
    webhook_path = stripe_event_path(conn, :create)

    case request_path do
      ^webhook_path -> verify_req(conn)
      _ -> conn
    end
  end

  defp verify_req(conn) do
    case read_body(conn) do
      {:ok, body, conn} -> do_verify(conn, body)
      {:more, _, conn} -> {:error, :too_large, conn}
    end
  end

  defp do_verify(conn, body) do
    [signature] = get_req_header(conn, "stripe-signature")

    body
    |> Stripe.Webhook.construct_event(signature, @secret)
    |> case do
      {:ok, %Stripe.Event{} = event} ->
        conn
        |> assign(:event, event)

      {:error, err} ->
        Logger.error("Error verifying Stripe Event: #{err}")

        conn
        |> send_resp(:bad_request, "Webhook Not Verified")
        |> halt()
    end
  end
end
{% endhighlight %}
