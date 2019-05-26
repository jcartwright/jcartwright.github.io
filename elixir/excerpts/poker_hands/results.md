{% highlight shell %}
poker_hands master % mix run benchmark.exs

Operating System: macOS
CPU Information: Intel(R) Core(TM) i7-6820HQ CPU @ 2.70GHz
Number of Available Cores: 8
Available memory: 16 GB
Elixir 1.8.1
Erlang 21.3

Benchmark suite executing with the following configuration:
warmup: 2 s
time: 5 s
memory time: 0 ns
parallel: 1
inputs: none specified
Estimated total run time: 28 s

Benchmarking run...
Benchmarking run_async...
Benchmarking stream...
Benchmarking stream_async...

Name                   ips        average  deviation         median         99th %
stream_async          0.24         4.22 s     ±0.61%         4.22 s         4.23 s
run_async             0.23         4.36 s     ±1.90%         4.36 s         4.42 s
stream               0.195         5.13 s    ±14.11%         5.13 s         5.65 s
run                  0.192         5.20 s     ±0.00%         5.20 s         5.20 s

Comparison:
stream_async          0.24
run_async             0.23 - 1.03x slower +0.147 s
stream               0.195 - 1.22x slower +0.92 s
run                  0.192 - 1.23x slower +0.99 s
{% endhighlight %}
