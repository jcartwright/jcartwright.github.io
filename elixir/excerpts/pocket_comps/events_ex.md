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
  Gets an event by id
  """
  def get_event(id) when is_binary(id) do
    Event.retrieve(id)
  end

  @doc """
  Gets an event for the given Stripe.Event
  """
  def get_event(%Event{id: id}), do: get_event(id)

  @doc """
  Handle an event based on id and type
  """
  def handle_event(%Event{id: id}) do
    ## Retrieve the event from Stripe to ensure it's legit
    with {:ok, %Event{} = event} <- get_event(id),
         :ok <- handle_event(event.type, event)
    do
      Logger.info("Handled Event[#{event.id}]: #{event.type}")
      {:ok, event.id}
    else
      {:error, %Stripe.Error{} = error} ->
        if Mix.env != :test do
          Logger.warn(error.message)
        end
        {:error, error}
      _ -> :noop
    end
  end

  defp handle_event("customer.subscription.created", event) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Subscription{} = subscription} <- get_object_for_event(event)
    do
      Mailer.trial_started_email(user, subscription)
      |> Mailer.deliver_later()

      :ok
    end
  end

  defp handle_event("customer.subscription.deleted", event) do
    with {:ok, %User{} = user} <- get_user_for_event(event) do
      user
      |> Mailer.canceled_email()
      |> Mailer.deliver_later()

      {:ok, _customer} = Customers.delete_customer(user)
      {:ok, _user} = Accounts.set_user_deleted(user)

      :ok
    end
  end

  defp handle_event("customer.subscription.trial_will_end", event) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Subscription{} = subscription} <- get_object_for_event(event)
    do
      # Unless the subscription has already ended,
      # notify the user that their trial will end...
      unless subscription.ended_at do
        Mailer.trial_will_end_email(user, subscription)
        |> Mailer.deliver_later()
      end

      :ok
    end
  end

  defp handle_event("customer.subscription.updated", event) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Subscription{} = subscription} <- get_object_for_event(event)
    do
      if subscription.cancel_at_period_end do
        Mailer.cancelation_email(user, subscription)
        |> Mailer.deliver_later()
      end
    end

    :ok
  end

  defp handle_event("invoice.payment_failed", event) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Invoice{} = invoice} <- get_object_for_event(event)
    do
      Mailer.payment_failed_email(user, invoice)
      |> Mailer.deliver_later()

      :ok
    end
  end

  defp handle_event("invoice.payment_succeeded", event) do
    with {:ok, %User{} = user} <- get_user_for_event(event),
         {:ok, %Invoice{} = invoice} <- get_object_for_event(event)
    do
      if invoice.paid and invoice.total > 0 do
        Mailer.payment_succeeded_email(user, invoice)
        |> Mailer.deliver_later()
      end

      :ok
    end
  end

  defp handle_event(_, event) do
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
