{% highlight elixir %}
## apps/mindbody_affiliate/lib/locations/locations.ex

defmodule MindbodyAffiliate.Locations do
  @moduledoc """
  Boundary for Affiliate Locations.
  """

  alias MindbodyAffiliate.Paginator
  import MindbodyAffiliate.Request

  @endpoint "locations"

  @doc """
  Returns a list of all locations.

  ### Examples:

    iex> MindbodyAffiliate.Locations.list
    {:ok, %{...}}

    iex> MindbodyAffiliate.Locations.list(%{searchText: "bikram"})
    {:ok, %{...}}

    iex> MindbodyAffiliate.Locations.list(%{subscriberId: 591})
    {:ok, %{...}}

  """
  def list(params \\ %{}, paginator \\ %Paginator{}) do
    paginator
    |> new_request()
    |> put_method(:get)
    |> put_endpoint(@endpoint)
    |> put_params(params)
    |> make_request()
    |> case do
      {:ok, %{"items" => locations} = response} ->
        next_paginator = Paginator.next_paginator(paginator, response)
        {:ok, locations, next_paginator}

      {:error, error} ->
        {:error, error}
    end
  end

  @doc """
  Returns a specific location by id.

  ### Examples:

    iex> MindbodyAffiliate.Locations.get(496560)
    {:ok, %{...}}

  """
  def get(id) do
    endpoint = resource_path(@endpoint, id)

    new_request()
    |> put_method(:get)
    |> put_endpoint(endpoint)
    |> make_request()
  end

  defp resource_path(endpoint, id),
    do: endpoint <> "/#{id}"
end
{% endhighlight %}
