{% highlight elixir %}
## lib/comps_api/properties/core_logic.ex

defmodule CompsAPI.Properties.CoreLogic do
  import SweetXml

  alias CompsAPI.Properties.{
    Property,
    Address,
    SquareFootage,
    Building
  }

  @config Application.get_env(:comps_api, __MODULE__)

  def get_property(address) do
    params = %{
      "address" => address |> URI.encode_www_form(),
      # ...
    }

    recv_timeout = @config[:recv_timeout] || 5_000

    @config[:base_url]
    |> HTTPoison.get([], params: params, recv_timeout: recv_timeout)
    |> case do
      {:ok, %{status_code: 200, body: body}} ->
        cond do
          xpath(body, ~x"(/PROPERTIES/PROPERTY)[1]") ->
            {:ok, body |> to_property()}

          true ->
            {:error, body |> xpath(~x"/message/text()"s)}
        end

      {:error, reason} ->
        {:error, reason}
    end
  end

  defp to_property(data) do
    %Property{
      address: build_address(data),
      square_footage: build_sqft(data),
      building: build_building(data)
    }
  end

  defp build_address(data) do
    address =
      data
      |> xpath(
        ~x"(/PROPERTIES/PROPERTY)[1]",
        key: ~x"./APN/text()"s,
        street_number: ~x"./PROPERTY_HOUSE_NUMBER/text()"s,
        street_prefix: ~x"./PROPERTY_PRE_DIRECTION/text()"s,
        street_name: ~x"./PROPERTY_STREET_NAME/text()"s,
        street_suffix: ~x"./PROPERTY_POST_DIRECTION/text()"s,
        street_mode: ~x"./PROPERTY_ADDRESS_MODE/text()"s,
        city: ~x"./MUNICIPALITY_NAME/text()"s,
        county: ~x"./COUNTY/text()"s,
        state: ~x"./PROPERTY_STATE_NAME/text()"s,
        postal: ~x"./PROPERTY_ZIP_CODE/text()"s,
        plus4: ~x"./PROPERTY_ZIP4/text()"s,
        subdivision: ~x"./SUBDIVISION_NAME/text()"s,
        latitude: ~x"./LATITUDE/text()"F,
        longitude: ~x"./LONGITUDE/text()"F
      )

    address
    |> Map.put(:street, build_street(address))
    |> Address.from_map()
  end

  defp build_building(data) do
    data
    |> xpath(
      ~x"(/PROPERTIES/PROPERTY)[1]",
      year_built: ~x"./YEAR_BUILT/text()"i,
      bedrooms: ~x"./BEDROOMS/text()"I,
      bathrooms: ~x"./TOTAL_BATHS/text()"I,
      full_baths: ~x"./FULL_BATHS/text()"I,
      half_baths: ~x"./HALF_BATHS/text()"I
    )
    |> Building.from_map()
  end

  defp build_sqft(data) do
    data
    |> xpath(
      ~x"(/PROPERTIES/PROPERTY)[1]",
      universal_building: ~x"./BUILDING_SQ_FT/text()"s
      |> transform_by(&(square_feet_to_integer(&1)))
    )
    |> SquareFootage.from_map()
  end

  defp build_street(%{
    street_number: number,
    street_prefix: prefix,
    street_name: name,
    street_suffix: suffix,
    street_mode: mode
  }) do
    "#{number} #{prefix} #{name} #{suffix} #{mode}"
    |> String.replace("  ", " ")
    |> String.trim()
  end

  defp square_feet_to_integer(building_sqft)
  when is_binary(building_sqft) do
    building_sqft
    |> String.replace(",", "")
    |> Integer.parse()
    |> case do
      {value, _} -> value
      :error -> 0
    end
  end

  defp square_feet_to_integer(_), do: 0
end
{% endhighlight %}
