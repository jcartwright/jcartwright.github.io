{% highlight elixir %}
## lib/comps_api/properties/homeland.ex

defmodule CompsAPI.Properties.Homeland do

  alias CompsAPI.Properties.Property
  alias CompsAPI.Properties.SquareFootage, as: SqFt

  @config Application.get_env(:comps_api, __MODULE__)

  def get_comps(%Property{} = property, opts \\ %{}) do
    {:ok, params} = build_params(property, opts)

    recv_timeout = @config[:recv_timeout] || 10_000

    @config[:comps_url]
    |> HTTPoison.get([], params: params, recv_timeout: recv_timeout)
    |> case do
      {:ok, %{status_code: 200, body: body}} ->
        case body |> Poison.decode! do
          %{"main" => _} = data ->
            decorated =
              data
              |> create_ids_for_comps(["sold_properties", "primary"])
              |> create_ids_for_comps(["rent_properties", "primary"])

            {:ok, property.comparables |> put_in(decorated)}

          %{"error" => true, "error-type" => message} ->
            {:error, message}
        end

      {:error, %{reason: reason}} ->
        {:error, reason}
    end
  end

  defp create_ids_for_comps(raw_comps, keys) do
    cond do
      length(raw_comps |> get_in(keys)) > 0 ->
        raw_comps
        |> update_in(keys ++ [Access.all()], &(put_in(&1["id"], comp_id(&1))))
      true -> raw_comps
    end
  end

  defp comp_id(%{ "status" => status, "status_date" => status_date,
                  "street_number" => number, "street_name" => name,
                  "street_suffix" => suffix }) do
    :crypto.hash(:sha,
                 [status, status_date, number, name, suffix]
                 |> Enum.filter(&(&1))
    ) |> Base.encode16
  end

  # ...

end
{% endhighlight %}
