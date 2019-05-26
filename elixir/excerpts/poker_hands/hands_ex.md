{% highlight elixir %}
## lib/hand.ex

defmodule PokerHands.Hand do
  alias PokerHands.Card

  def score_hand(hand) when is_binary(hand) do
    hand
    |> String.split()
    |> score_hand()
  end

  def score_hand(hand) when is_list(hand) do
    cards =
      hand
      |> Enum.map(&Card.parse/1)
      |> sort_cards()

    cond do
      royal_flush?(cards) ->
        {:royal_flush, cards}

      straight_flush?(cards) ->
        {:straight_flush, cards}

      four_of_a_kind?(cards) ->
        {:four_of_a_kind, cards}

      full_house?(cards) ->
        {score, _set} =
          cards
          |> get_sets()
          |> List.first()

        {:full_house, score}

      flush?(cards) ->
        {:flush, cards}

      straight?(cards) ->
        {:straight, cards}

      three_of_a_kind?(cards) ->
        {:three_of_a_kind, cards}

      two_pair?(cards) ->
        {:two_pair, cards}

      one_pair?(cards) ->
        {score, _set} =
          cards
          |> get_pairs()
          |> List.first()

        rem =
          cards
          |> Enum.reject(fn {face, _suit} -> face == score end)
          |> Enum.reduce(0, fn {face, _suit}, val -> val + face end)

        {:one_pair, score + rem / 100}

      true ->
        {:high_card, cards}
    end
  end

  #######################################
  ## PokerHands.Hand ranking functions ##
  #######################################

  defp royal_flush?(cards) do
    royal?(cards) and flush?(cards) and sequential?(cards)
  end

  defp straight_flush?(cards) do
    flush?(cards) and sequential?(cards)
  end

  defp four_of_a_kind?(cards) do
    cards
    |> group_sets()
    |> Enum.any?(fn {_, set} -> length(set) == 4 end)
  end

  defp full_house?(cards) do
    three_of_a_kind?(cards) and one_pair?(cards)
  end

  defp flush?(cards) do
    1 ==
      cards
      |> Enum.uniq_by(fn {_, suit} -> suit end)
      |> Enum.count()
  end

  defp straight?(cards) do
    cards
    |> sequential?()
  end

  defp three_of_a_kind?(cards) do
    cards
    |> group_sets()
    |> Enum.any?(fn {_, set} -> length(set) == 3 end)
  end

  defp two_pair?(cards),
    do: 2 == cards |> count_pairs()

  defp one_pair?(cards),
    do: 1 == cards |> count_pairs()

  ######################################
  ## PokerHands.Hand helper functions ##
  ######################################

  defp count_pairs(cards) do
    cards
    |> group_sets()
    |> Enum.count(fn {_, set} -> length(set) == 2 end)
  end

  defp get_pairs(cards) do
    cards
    |> group_sets()
    |> Enum.filter(fn {_, set} -> length(set) == 2 end)
  end

  defp get_sets(cards) do
    cards
    |> group_sets()
    |> Enum.sort(&(length(elem(&1, 1)) > length(elem(&2, 1))))
  end

  defp group_sets(cards) do
    cards
    |> Enum.group_by(fn {face, _suit} -> face end)
  end

  defp royal?(cards) do
    cards
    |> Enum.all?(fn {face, _suit} -> face in Card.royals() end)
  end

  defp sort_cards(cards) do
    cards
    |> Enum.sort(&(elem(&1, 0) > elem(&2, 0)))
  end

  defp sequential?(cards) do
    unique_values =
      cards
      |> Enum.uniq_by(fn {face, _suit} -> face end)

    5 == length(unique_values) and 5 == run_of(unique_values)
  end

  defp run_of(sorted_cards), do: run_of(sorted_cards, 1)

  defp run_of([{face, _} | tail], length) do
    run_of(tail, face, length)
  end

  defp run_of([], _, length), do: length

  defp run_of([{face, _} | tail], prev, length) do
    if prev - face == 1 do
      run_of(tail, face, length + 1)
    else
      run_of(tail, face, length)
    end
  end
end
{% endhighlight %}
