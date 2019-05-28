---
title: Elixir |> Portfolio
permalink: /elixir/portfolio.html
---

Since a majority of the Elixir code I've written is in private repos I've pulled
together some excerpts and examples to illustrate the more interesting bits of
my recent work.

* [OSS Contribution to stripity_stripe](#stripity_stripe) May 2019 - _Present_
* [PocketComps API](#pocket_comps) June 2017 - _Present_
* [MINDBODY API to CSV](#mindbody) Feb 2019 - April 2019
* [Project Euler - Poker Hands](#poker_hands) March 2019
* [Yatzy-Refactoring-Kata Elixir](#yatzy_refactor) April 2019

<a id="#stripity_stripe"></a>OSS Contribution to [stripity_stripe][1]
---

[stripity_stripe][1] is an open source hex package that wraps Stripe's API for
Elixir. I've used it in the [PocketComps](#pocket_comps) application for
recurring subscription payment processing. I'm now leveraging it for a very
different use case that requires [Stripe's Issuing Service][10], which issues
and manages physical cards and virtual cards for proxy payments.

Since Stripe Issuing is a fairly new service, the existing [stripity_stripe][1]
package does not have support for it. Rather than creating my own wrapper, I
prefer to leverage an existing open-source option, and contribute that work to
the community.

I've got ~~an open~~ a merged [pull request][2] that adds support for
Stripe Issuing to the stripity_stripe package. This illustrates how I
collaborate with the maintainers and work with existing codebases to match the
style and organization. Here is an excerpt from my pull request.

<a id="issuing_card"></a>
{% include_relative excerpts/card_ex_short.md %}

<a id="card_test"></a>
{% include_relative excerpts/card_test_exs.md %}

<br/>

<a id="pocket_comps"></a>PocketComps API
---

[PocketComps][3] is a mobile-first application I've built using [Vue.js][4] and
Elixir that provides simple, but powerful access to Real Estate CMAs for
Investors and Agents. The API leverages several 3rd party APIs and aggregates
the data to provide users with information necessary to evaluate the potential
value of a residential property.

PocketComps uses Stripe for recurring subscriptions. The following excerpts show
the key parts of the Stripe Webhook event handling. When webhook events hit the
[`event_controller`](#event_controller), they are backgrounded using
`Task.Supervisor.async_nolink/1` and a `200 OK` is returned immediately to Stripe.

<a id="pocket_comps_router"></a>
{% include_relative excerpts/pocket_comps/router_ex.md %}

<a id="verify_plug"></a>
{% include_relative excerpts/pocket_comps/verify_plug_ex.md %}

<a id="event_controller"></a>
{% include_relative excerpts/pocket_comps/event_controller_ex.md %}

<a id="events_handler"></a>
{% include_relative excerpts/pocket_comps/events_ex.md %}

One of the 3rd party data providers returns XML as the response. I'm using [SweetXml][5] to parse the XML response and their `~x` sigil to extract and transform data via xpath.

<a id="core_logic"></a>
{% include_relative excerpts/pocket_comps/core_logic_ex.md %}

An additional bit of "interesting" code is where I need to build a unique
identifier for each property returned. The source data does not include a unique
id, and properties can exist in multiple statuses, so the address itself is not
guaranteed to be unique.

The following excerpt shows how I used the [`Access`][6] module to generate and
put unique identifiers into lists of nested data. Particularly in the
`create_ids_for_comps` function there's a combination of the `update_in`,
`Access.all()`. and `put_in` functions that perform the task concisely, albeit
somewhat cryptically.

<a id="homeland"></a>
{% include_relative excerpts/pocket_comps/homeland_ex.md %}

<br/>

<a id="mindbody"></a>MINDBODY API to CSV
---

MINDBODY is the leading provider of integrated fitness studios across the
country. They have recently launched an "affiliate" API which provides over
11,000 locations in the US. In order to analyze the details of the affiliate
locations and their impact on network expansion, I began building the
integration with Elixir and export the data to CSV.

The `locations` endpoint supports pagination and a maximum page size of 1000.
The API also enforces rate limiting on multiple axes including daily totals,
requests per second, and spike arrest of concurrent requests less than 100ms
apart.

The code presented here is not for production, and is a utility to efficiently
export the entire network for analysis. For each of the 11k plus locations,
there are 20 to 100 classes to export and additional supporting data, each
requiring separate API calls.

What I think are the interesting parts are the CSV export libraries and the use
of Elixir's `Stream.resource/3` in the `locations_stream/0` function to stream
the paginated results into further API calls and final aggregation of the data
to export.

<a id="exporter_ex"></a>
{% include_relative excerpts/mindbody/exporter_ex.md %}

<a id="paginator_ex"></a>
{% include_relative excerpts/mindbody/paginator_ex.md %}

<a id="request_ex"></a>
{% include_relative excerpts/mindbody/request_ex.md %}

<a id="locations_ex"></a>
{% include_relative excerpts/mindbody/locations_ex.md %}

<a id="classes_ex"></a>
{% include_relative excerpts/mindbody/classes_ex.md %}


<br/>

<a id="poker_hands"></a>Project Euler - [Poker Hands][7]
---

Project Euler is a site with scores of mathematical and programmatic challenges.
I frequently user [Poker Hands][7] as a take-home code exercise in my interview
process because it is a fun and reasonably scoped project that most developers
can complete in less than 5 hours.

I created this solution for Elixir as part of the team training I lead and as a
baseline for future candidate submissions. The additional twist I added was to
compare four different ways to execute the task using `run` (sync), `run_async`,
`stream` (sync) and `stream_async` with [Benchee][8] to measure the different
performance characteristics of each. To get any benefit from the streaming
options we had to duplicate the provided 1000-line `poker_hands.txt` file to
100,000 lines. The following is the output of the benchmark.

{% include_relative excerpts/poker_hands/results.md %}

The main module is `PokerHands`, which the `benchmark.exs` file references when
setting up the benchmarks to run.

<a id="poker_hands_ex"></a>
{% include_relative excerpts/poker_hands/poker_hands_ex.md %}

<a id="hand_ex"></a>
{% include_relative excerpts/poker_hands/hands_ex.md %}

<a id="card_ex"></a>
{% include_relative excerpts/poker_hands/card_ex.md %}

<br/>

<a id="yatzy_refactor"></a>Yatzy-Refactoring-Kata [Elixir][9]
---

The Yatzy Refactoring Kata is another tool I use in technical interviews.
Candidates will pair-program with members of the team to refactor some very
badly written code. The interesting challenge for me in creating the Elixir
version was to find egregious ways to write functions that worked and passed the
tests. And note, even the tests are an intentional mess.

I've included this just for fun and as another illustration of ways I'm
contributing back to the Elixir community.

**Please don't judge me on this code!**

<a id="test_yatzy_exs"></a>
{% include_relative excerpts/yatzy/test_yatzy_exs.md %}

<a id="yatzy_ex"></a>
{% include_relative excerpts/yatzy/yatzy_ex.md %}



[1]: https://github.com/code-corps/stripity_stripe
[2]: https://github.com/code-corps/stripity_stripe/pull/493 "Add Stripe Issuing"
[3]: https://www.pocketcomps.com
[4]: https://vuejs.org
[5]: https://github.com/kbrw/sweet_xml "SweetXml"
[6]: https://hexdocs.pm/elixir/Access.html
[7]: https://projecteuler.net/problem=54
[8]: https://github.com/bencheeorg/benchee
[9]: https://github.com/emilybache/Yatzy-Refactoring-Kata/tree/master/elixir
[10]: https://stripe.com/docs/issuing
