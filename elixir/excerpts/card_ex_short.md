{% highlight elixir %}
## lib/stripe/issuing/card.ex

defmodule Stripe.Issuing.Card do
  @moduledoc """
  Work with Stripe Issuing card objects.

  You can:

  - Create a card
  - Retrieve a card
  - Update a card
  - List all cards

  Stripe API reference: https://stripe.com/docs/api/issuing/cards
  """

  use Stripe.Entity
  import Stripe.Request

  @type t :: %__MODULE__{
          id: Stripe.id(),
          object: String.t(),
          authorization_controls: Stripe.Issuing.Types.authorization_controls(),
          brand: String.t(),
          cardholder: Stripe.Issuing.Cardholder.t(),
          created: Stripe.timestamp(),
          currency: String.t(),
          exp_month: pos_integer,
          exp_year: pos_integer,
          last4: String.t(),
          livemode: boolean,
          metadata: Stripe.Types.metadata(),
          name: String.t(),
          replacement_for: t | Stripe.id() | nil,
          replacement_reason: String.t() | nil,
          shipping: Stripe.Types.shipping() | nil,
          status: String.t(),
          type: atom() | String.t()
        }

  defstruct [
    :id,
    :object,
    :authorization_controls,
    :brand,
    :cardholder,
    :created,
    :currency,
    :exp_month,
    :exp_year,
    :last4,
    :livemode,
    :metadata,
    :name,
    :replacement_for,
    :replacement_reason,
    :shipping,
    :status,
    :type
  ]

  @plural_endpoint "issuing/cards"

  @doc """
  Create a card.
  """
  @spec create(params, Stripe.options()) :: {:ok, t} | {:error, Stripe.Error.t()}
        when params:
               %{
                 :currency => String.t(),
                 :type => :physical | :virtual,
                 optional(:authorization_controls) =>
                   Stripe.Issuing.Types.authorization_controls(),
                 optional(:cardholder) => Stripe.Issuing.Cardholder.t(),
                 optional(:metadata) => Stripe.Types.metadata(),
                 optional(:replacement_for) => t | Stripe.id(),
                 optional(:replacement_reason) => String.t(),
                 optional(:shipping) => Stripe.Types.shipping(),
                 optional(:status) => String.t()
               }
               | %{}
  def create(params, opts \\ []) do
    new_request(opts)
    |> put_endpoint(@plural_endpoint)
    |> put_params(params)
    |> put_method(:post)
    |> make_request()
  end

  # ...

end
{% endhighlight %}
