{% highlight elixir %}
## lib/comps_api/payments/events.ex

defmodule CompsAPI.Payments.Events do
  @moduledoc """
  The boundary for the Payments.Events system
  """

  require Logger

  alias Stripe.{ Event, Invoice, Subscription }
  alias CompsAPI.Accounts
  alias CompsAPI.Accounts.User
  alias CompsAPI.Payments.Customers
  alias CompsAPI.SendGrid.Mailer

  @doc """
  Handle a Stripe.Event
  """
  def handle_event(%Event{id: event_id}) do
    ## Retrieve the event from Stripe to ensure it's legit
    with {:ok, %Event{} = event} <- get_event(event_id) do
      case process_event(event) do
        {:ok, processed_event} ->
          Logger.info("Handled Event[#{processed_event.id}]: #{processed_event.type}")
          {:ok, processed_event}

        _ -> :noop
      end
    else
      {:error, %Stripe.Error{} = error} ->
        if Mix.env != :test do
          Logger.warn(error.message)
        end
        {:error, error}
    end
  end

  def get_event(%Event{id: event_id}),
    do: get_event(event_id)

  def get_event(event_id) when is_binary(event_id),
    do: Event.retrieve(event_id)

  defp process_event(event = %{type: "customer.subscription.created"}) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Subscription{} = subscription} <- get_object_for_event(event)
    do
      Mailer.trial_started_email(user, subscription)
      |> Mailer.deliver_later()

      {:ok, event}
    end
  end

  defp process_event(event = %{type: "customer.subscription.deleted"}) do
    with {:ok, %User{} = user} <- get_user_for_event(event) do
      user
      |> Mailer.canceled_email()
      |> Mailer.deliver_later()

      {:ok, _customer} = Customers.delete_customer(user)
      {:ok, _user} = Accounts.set_user_deleted(user)

      {:ok, event}
    end
  end

  defp process_event(event = %{type: "customer.subscription.trial_will_end"}) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Subscription{} = subscription} <- get_object_for_event(event)
    do
      # Unless the subscription has already ended,
      # notify the user that their trial will end...
      unless subscription.ended_at do
        Mailer.trial_will_end_email(user, subscription)
        |> Mailer.deliver_later()
      end

      {:ok, event}
    end
  end

  defp process_event(event = %{type: "customer.subscription.updated"}) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Subscription{} = subscription} <- get_object_for_event(event)
    do
      if subscription.cancel_at_period_end do
        Mailer.cancelation_email(user, subscription)
        |> Mailer.deliver_later()
      end
    end

    {:ok, event}
  end

  defp process_event(event = %{type: "invoice.payment_failed"}) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Invoice{} = invoice} <- get_object_for_event(event)
    do
      Mailer.payment_failed_email(user, invoice)
      |> Mailer.deliver_later()

      {:ok, event}
    end
  end

  defp process_event(event = %{type: "invoice.payment_succeeded"}) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Invoice{} = invoice} <- get_object_for_event(event)
    do
      if invoice.paid and invoice.total > 0 do
        Mailer.payment_succeeded_email(user, invoice)
        |> Mailer.deliver_later()
      end

      {:ok, event}
    end
  end

  defp process_event(event) do
    Logger.info("Unhandled Event[#{event.id}]: #{event.type}")
    :noop
  end

  defp get_object_for_event(event) do
    with {:ok, data} <- Map.fetch(event, :data),
         {:ok, object} <- Map.fetch(data, :object)
    do
      {:ok, object}
    end
  end

  defp get_user_for_event(event) do
    with {:ok, object}  <- get_object_for_event(event),
         %User{} = user <- object.customer
                           |> Accounts.get_user_by_customer
    do
      {:ok, user}
    end
  end

end
{% endhighlight %}
