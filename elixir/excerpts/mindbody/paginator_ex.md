{% highlight elixir %}
## apps/mindbody_affiliate/lib/paginator.ex

defmodule MindbodyAffiliate.Paginator do
  @moduledoc """
  Provides a standardized struct for building and advancing
  pagination in the API endpoints that support it.
  """

  alias __MODULE__

  @type t :: %__MODULE__{
    max_results: non_neg_integer,
    offset: non_neg_integer,
    total_results: non_neg_integer
  }

  defstruct max_results: 500,
            offset: 0,
            total_results: 0

  @spec to_params(t) :: map
  def to_params(paginator = %Paginator{}) do
    %{
      "maxResults" => paginator.max_results,
      "offset" => paginator.offset,
      "totalResults" => paginator.total_results
    }
  end

  def next_paginator(_paginator, %{"totalResults" => 0}),
    do: nil

  def next_paginator(paginator, %{
        "totalResults" => total_results,
        "offset" => current_offset,
        "maxResults" => max_results
      }) do
    next_offset = current_offset + max_results

    if total_results > next_offset do
      %{paginator | offset: next_offset, total_results: total_results}
    end
  end

  def next_paginator(_, _), do: nil
end
{% endhighlight %}
