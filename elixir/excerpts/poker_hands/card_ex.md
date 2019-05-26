{% highlight elixir %}
## lib/card.ex

defmodule PokerHands.Card do
  @face_values %{
    "T" => 10,
    "J" => 11,
    "Q" => 12,
    "K" => 13,
    "A" => 14
  }

  def parse(card) do
    val = fn -> face_value(card) end
    suit = fn -> suit(card) end

    {val.(), suit.()}
  end

  def royals do
    @face_values
    |> Map.values()
  end

  defp face_value(card) do
    value =
      card
      |> String.at(0)

    value
    |> Integer.parse()
    |> case do
      {val, _} -> val
      :error -> @face_values[value]
    end
  end

  defp suit(card) do
    card
    |> String.at(1)
  end
end
{% endhighlight %}
