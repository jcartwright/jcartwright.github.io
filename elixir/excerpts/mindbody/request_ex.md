{% highlight elixir %}
## apps/mindbody_affiliate/lib/request.ex

defmodule MindbodyAffiliate.Request do
  @moduledoc """
  Used as a token for building up a request to the
  MINDBODY Affiliate API.
  """

  defstruct endpoint: nil,
            headers: nil,
            method: nil,
            paginator: nil,
            params: %{}

  alias __MODULE__
  alias MindbodyAffiliate.Paginator

  def new_request(paginator \\ %Paginator{}, headers \\ %{}) do
    %Request{paginator: paginator, headers: headers}
  end

  def put_endpoint(request = %Request{}, endpoint) do
    %{request | endpoint: endpoint}
  end

  def put_method(request = %Request{}, method) do
    %{request | method: method}
  end

  def put_params(request = %Request{}, params) do
    %{request | params: params}
  end

  def make_request(%Request{
        endpoint: endpoint,
        headers: headers,
        method: method,
        paginator: paginator,
        params: params
      }) do
    with {:ok, params} <- merge_paginator_and_params(paginator, params),
         {:ok, headers} <- merge_headers(headers),
         {:ok, endpoint} <- build_endpoint(endpoint) do
      method
      |> case do
        :get ->
          endpoint
          |> HTTPoison.get(headers, [params: params, recv_timeout: 10_000])
          |> decode_response()
      end
    end
  end

  defp config,
    do: Application.fetch_env!(:mindbody_affiliate, MindbodyAffiliate)

  defp build_endpoint(endpoint) do
    {:ok, config()[:base_url] <> endpoint}
  end

  defp default_headers(headers) do
    %{
      "content-type": "application/json",
      "api-key": config()[:api_key],
      authorization: "Basic #{encoded_source_data()}"
    }
    |> Map.merge(headers)
  end

  defp encoded_source_data do
    Base.encode64("#{config()[:client_key]}:#{config()[:client_secret]}")
  end

  defp merge_paginator_and_params(paginator, params) do
    {:ok,
     paginator
     |> Paginator.to_params()
     |> Map.merge(params)}
  end

  defp merge_headers(headers) do
    {:ok,
     headers
     |> default_headers()}
  end

  defp decode_response({:ok, response}) do
    case response do
      %{status_code: status_code, body: body}
      when status_code in 200..201 ->
        {:ok, body |> Poison.decode!()}

      %{status_code: 401} ->
        {:error, "401 Unauthorized"}

      %{status_code: 404, body: body} ->
        {:error, body |> Poison.decode!()}

      %{status_code: 429} ->
        {:error, "429 Rate Limit Exceeded"}
    end
  end

  defp decode_response({:error, error}) do
    {:error, error}
  end
end
{% endhighlight %}
