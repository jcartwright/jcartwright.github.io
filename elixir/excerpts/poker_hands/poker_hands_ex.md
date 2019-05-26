{% highlight elixir %}
## lib/poker_hands.ex

defmodule PokerHands do

  alias PokerHands.Hand

  @game_file "poker_hands_large.txt"

  @ranks [
    :high_card,
    :one_pair,
    :two_pair,
    :three_of_a_kind,
    :straight,
    :flush,
    :full_house,
    :four_of_a_kind,
    :straight_flush,
    :royal_flush
  ]

  def run() do
    File.read!(@game_file)
    |> String.split("\n", trim: true)
    |> Enum.map(&parse_line/1)
    |> Enum.map(&play_hand/1)
    |> calculate_wins()
  end

  def run_async() do
    File.read!(@game_file)
    |> String.split("\n", trim: true)
    |> Task.async_stream(fn line ->
      line
      |> parse_line()
      |> play_hand()
    end)
    |> calculate_async_wins()
  end

  def stream() do
    File.stream!(@game_file)
    |> Stream.map(&parse_line/1)
    |> Stream.map(&play_hand/1)
    |> calculate_wins()
  end

  def stream_async() do
    File.stream!(@game_file)
    |> Task.async_stream(fn line ->
      line
      |> parse_line()
      |> play_hand()
    end)
    |> calculate_async_wins()
  end

  def play_hand(p1, p2), do: play_hand([p1, p2])

  def play_hand([p1, p2]) do
    rank = &Enum.find_index(@ranks, fn rank -> rank == &1 end)

    {p1_rank, p1_score} = Hand.score_hand(p1)
    {p2_rank, p2_score} = Hand.score_hand(p2)

    rank1 = rank.(p1_rank)
    rank2 = rank.(p2_rank)

    cond do
      rank1 > rank2 -> :p1
      rank2 > rank1 -> :p2
      true -> handle_tie(p1_score, p2_score)
    end
  end

  defp calculate_async_wins(results) do
    results
    |> Enum.group_by(
      fn {:ok, player} -> player end,
      fn {:ok, player} -> player end
    )
    |> Enum.map(fn {player, wins} ->
      {player, length(wins)}
    end)
  end

  defp calculate_wins(results) do
    results
    |> Enum.group_by(fn player -> player end)
    |> Enum.map(fn {player, wins} ->
      {player, length(wins)}
    end)
  end

  defp handle_tie(p1, p2) do
    cond do
      p1 > p2 -> :p1
      p2 > p1 -> :p2
      true -> :tie
    end
  end

  defp parse_line(line) do
    line
    |> String.split_at(15)
    |> Tuple.to_list()
  end
end
{% endhighlight %}
