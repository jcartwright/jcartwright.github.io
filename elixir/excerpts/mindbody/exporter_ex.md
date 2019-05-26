{% highlight elixir %}
## apps/mindbody_affiliate/lib/exporter.ex

NimbleCSV.define(CSVParser, separator: "\t", escape: "\")

defmodule MindbodyAffiliate.Exporter do
  @moduledoc """
  A Module for exporting MBO Affiliate Locations to CSV.
  """

  alias MindbodyAffiliate.{
    Classes,
    Locations,
    Paginator,
    PricingOptions
  }

  @bom :unicode.encoding_to_bom({:utf16, :little})
  @page_size 1000

  def classes_to_csv(filename \\ "classes.csv") do
    header =
      class_mapping()
      |> Keyword.keys()
      |> Enum.map(&to_string/1)

    data =
      locations_stream()
      |> Stream.map(&fetch_classes_for_location/1)
      |> Stream.flat_map(fn location ->
        location["classes"]
        |> Enum.map(fn class ->
          green_dot()

          for {_key, path} <- class_mapping() do
            class
            |> get_in(String.split(path, "."))
            |> to_string()
            |> String.trim()
          end
        end)
      end)
      |> Enum.to_list()

    filename
    |> File.write!([@bom, data_to_csv(header, data)])
  end

  def locations_to_csv(filename \\ "locations.csv") do
    header =
      location_mapping()
      |> Keyword.keys()
      |> Enum.map(&to_string/1)

    data =
      locations_stream()
      |> Stream.map(&fetch_mvp_for_location/1)
      |> Enum.map(fn item ->
        green_dot()

        for {_key, path} <- location_mapping() do
          item
          |> get_in(String.split(path, "."))
          |> to_string()
          |> String.trim()
        end
      end)

    filename
    |> File.write!([@bom, data_to_csv(header, data)])
  end

  def locations_stream do
    Stream.resource(
      fn -> %Paginator{max_results: @page_size} end,
      &process_page/1,
      fn _ -> :ok end
    )
  end

  defp process_page(nil) do
    {:halt, nil}
  end

  defp process_page(paginator = %Paginator{}) do
    paginator
    |> fetch_page()
  end

  defp fetch_page(paginator = %Paginator{}) do
    fetch_locations = &Locations.list(%{}, &1)

    paginator
    |> fetch_locations.()
    |> case do
      {:ok, locations, next_paginator} ->
        # return a tuple with the data and "next" paginator
        {locations, next_paginator}

      {:error, reason} ->
        IO.inspect(reason, label: "fetch_page")
        # TODO: handle rate limiting; For now we're just returning
        # an empty list and the same paginator as a "retry".
        {[], paginator}

      _ ->
        nil
    end
  end

  defp fetch_classes_for_location(location = %{"id" => location_id}) do
    params = %{}
    class_paginator = %Paginator{max_results: 100}

    with {:ok, classes, _} <- Classes.list(location_id, params, class_paginator) do
      location
      |> Map.put("classes", classes)
    else
      {:error, _reason} ->
        red_dot()
        Process.sleep(500)
        fetch_classes_for_location(location)
    end
  end

  @option_blacklist ~w(
    storage
    rental
    # ...
  )
  defp fetch_mvp_for_location(location = %{"id" => location_id}) do
    # Minimum Viable Pricing Option
    with {:ok, pricing_options} <- PricingOptions.list(location_id) do
      viable_dropin_options =
        pricing_options
        |> Enum.filter(&(&1["type"] == "DropIn"))
        |> Enum.reject(&(
          &1["name"]
          |> String.downcase()
          |> String.contains?(@option_blacklist)
        ))
        |> Enum.filter(&(&1["total"] >= 10 and &1["total"] <= 99))

      mvp_option =
        viable_dropin_options
        |> Enum.min_by(&(&1["total"]), fn -> nil end)

      location
      |> Map.put("minimumViablePricingOption", mvp_option)
      |> Map.put("viablePricingOptions",
        viable_dropin_options
        |> Enum.map(fn option ->
          "#{sanitize(option["name"])}@#{option["total"]}"
        end)
        |> Enum.join(", ")
      )
    else
      {:error, _reason} ->
        red_dot()
        Process.sleep(500)
        fetch_mvp_for_location(location)
    end
  end

  defp sanitize(text) do
    text
    |> String.trim()
    |> String.replace("'", "")
  end

  defp data_to_csv(data) do
    data
    |> CSVParser.dump_to_iodata()
    |> :unicode.characters_to_binary(:utf8, {:utf16, :little})
  end

  defp data_to_csv(header, data),
    do: data_to_csv([header] ++ data)

  defp green_dot do
    IO.ANSI.format([:green, :bright, "."], true)
    |> IO.write()
  end

  defp red_dot do
    IO.ANSI.format([:red, :bright, "."], true)
    |> IO.write()
  end

  defp class_mapping do
    # NOTE: We're using a Keyword List because the order
    # of the data is important in the output to CSV.
    [
      subscriber_id: "location.subscriber.id",
      subscriber_name: "location.subscriber.name",
      location_id: "location.id",
      location_name: "location.name",
      class_id: "id",
      name: "name",
      class_size: "classSize",
      available_capacity: "availableCapacity",
      start_date_time: "startDateTime",
      end_date_time: "endDateTime",
      price_low: "price.lowest",
      price_high: "price.highest"
    ]
  end

  defp location_mapping do
    # NOTE: We're using a Keyword List because the order
    # of the data is important in the output to CSV.
    [
      subscriber_id: "subscriber.id",
      subscriber_name: "subscriber.name",
      location_id: "id",
      location_name: "name",
      street: "contactInfo.streetAddress",
      city: "contactInfo.city",
      state: "contactInfo.stateCode",
      postal: "contactInfo.postalCode",
      country: "contactInfo.countryCode",
      latitude: "contactInfo.latitude",
      longitude: "contactInfo.longitude",
      pricing_option: "minimumViablePricingOption.name",
      price: "minimumViablePricingOption.total",
      viable_pricing_options: "viablePricingOptions"
    ]
  end
end
{% endhighlight %}
