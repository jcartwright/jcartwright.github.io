{% highlight elixir %}
##apps/mindbody_affiliate/lib/classes/classes.ex

defmodule MindbodyAffiliate.Classes do
  @moduledoc """
  Boundary for Affiliate Classes.
  """

  alias MindbodyAffiliate.Paginator
  import MindbodyAffiliate.Request

  @endpoint "locations/:location_id/classes"

  @doc """
  Returns a list of all classes for the specified location.

  ### Examples:

    iex> MindbodyAffiliate.Classes.list(496289)
    {:ok, %{...}}

    iex> MindbodyAffiliate.Classes.list(496289, %{availableForBooking: true})
    {:ok, %{...}}

  """
  def list(location_id, params \\ %{}, paginator \\ %Paginator{}) do
    endpoint = resource_path(@endpoint, location_id)

    paginator
    |> new_request()
    |> put_method(:get)
    |> put_endpoint(endpoint)
    |> put_params(params)
    |> make_request()
    |> case do
      {:ok, %{"items" => classes} = response} ->
        next_paginator = Paginator.next_paginator(paginator, response)
        {:ok, classes, next_paginator}

      {:error, error} ->
        {:error, error}
    end
  end

  @doc """
  Returns a specific class by location and id.

  ### Examples:

    iex> MindbodyAffiliate.Classes.get(496289, 209888)
    {:ok, %{...}}

  """
  def get(location_id, id) do
    endpoint = resource_path(@endpoint, location_id, id)

    new_request()
    |> put_method(:get)
    |> put_endpoint(endpoint)
    |> make_request()
  end

  defp resource_path(endpoint, location_id) do
    path = &String.replace(endpoint, ":location_id", &1)

    location_id
    |> to_string()
    |> path.()
  end

  defp resource_path(endpoint, location_id, id),
    do: resource_path(endpoint, location_id) <> "/#{id}"
end
{% endhighlight %}
