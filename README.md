# Setup

This project requires a `gcc` compiler installed and the `protobuf` code generation tools.

## Install protobuf compiler

Download the appropriate binary for your operating system from [https://github.com/protocolbuffers/protobuf/releases](https://github.com/protocolbuffers/protobuf/releases).

Install by placing the extracted binary somewhere in your `$PATH`. Additional instructions available at [https://grpc.io/docs/protoc-installation/](https://grpc.io/docs/protoc-installation/).


## Install Go protobuf codegen tools

`go install google.golang.org/protobuf/cmd/protoc-gen-go@latest`

`go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest`


## Generate Go code from .proto files

After creating your `.proto` file, this command will generate the necessary code.

```
protoc --go_out=. --go_opt=paths=source_relative \
  --go-grpc_out=. --go-grpc_opt=paths=source_relative \
  Proto/mail.proto
```
# Pixl Project Screenshot

<img width="1430" alt="Pixl" src="https://user-images.githubusercontent.com/22866744/211433774-899df70d-8adb-4a6d-b305-d8f426f1b641.png">

# Keiko Corp Image Server

The __Keiko Corp Image Server__ picks a random image from an image directory and serves it to the user. Whenever an image is served, it gets logged into a SQLite database.

## Assignment

Despite the Keiko Corp Image Server being written in Go, the performance isn't quite what Bruno was expecting. This is where you come in!

Make whatever changes you feel are necessary to improve the performance of the Keiko Corp Image Server. The server must:

* Serve images in the WebP format
* Track the number of hits per image
* Log the hits to the database

Once you have improved the performance, post your `requests per second` results to the `#go` channel in the ZTM Discord chat to see how your improvements compare to others!

## Runtime Requirements

The server utilizes [SQLite](https://www.sqlite.org/index.html) which requires `gcc` to be installed on your system.

## Benchmarking

[k6](https://k6.io/) is used to benchmark the performance of the server. To install `k6`, run `go install go.k6.io/k6@latest`.

After installation completes, you can run a benchmark with `k6 run bench.js`.

### Interpreting the Results

The benchmark results will look something like this:

```
running (32.1s), 0000/5000 VUs, 1262253 complete and 0 interrupted iterations
basic     ??? [======================================] 00/40 VUs      10s
high_load ??? [======================================] 000/200 VUs    10s
spike     ??? [======================================] 0000/5000 VUs  10s

     data_received..................: 119 MB  3.7 MB/s
     data_sent......................: 121 MB  3.8 MB/s
     http_req_blocked...............: avg=7.38ms  min=36.63??s  med=143.89??s max=279.68ms p(90)=29.81ms p(95)=37.96ms
     http_req_connecting............: avg=7.29ms  min=24.04??s  med=113.89??s max=200.76ms p(90)=29.62ms p(95)=37.63ms
     http_req_duration..............: avg=12.54ms min=45.38??s  med=1.28ms   max=250.59ms p(90)=43.82ms p(95)=53.52ms
       { expected_response:true }...: avg=12.54ms min=45.38??s  med=1.28ms   max=250.59ms p(90)=43.82ms p(95)=53.52ms
   ??? http_req_failed................: 0.00%   ??? 0           ??? 1262253
     http_req_receiving.............: avg=2.35ms  min=5.79??s   med=313.1??s  max=175.32ms p(90)=8.04ms  p(95)=10.79ms
     http_req_sending...............: avg=1.96ms  min=6.98??s   med=182.69??s max=84.67ms  p(90)=6.77ms  p(95)=9.57ms
     http_req_tls_handshaking.......: avg=0s      min=0s       med=0s       max=0s       p(90)=0s      p(95)=0s
     http_req_waiting...............: avg=8.21ms  min=20.57??s  med=391.6??s  max=192.97ms p(90)=31ms    p(95)=39.26ms
     http_reqs......................: 1262253 39382.05388/s
     iteration_duration.............: avg=20.68ms min=105.82??s med=1.55ms   max=294.06ms p(90)=76.56ms p(95)=94.82ms
     iterations.....................: 1262253 39382.05388/s
     vus............................: 4816    min=0         max=4816
     vus_max........................: 5000    min=5000      max=5000
```

Keiko Corp is interested in:

* `http_req_duration`: how long it takes to serve requests
* `http_reqs`: requests per second

90% of requests must be served within 100ms, 95% within 200ms, and 99% within 500ms. At least 99% of requests must succeed. The benchmark is configured to indicate if any of these metrics fail.

Since results will vary across machines, run the benchmark 

## Optimization Tips

Run a full benchmark (`k6 run bench.js`) prior to making any changes, and then save the results. This will allow you to measure how much of an impact your changes have to the performance of the application.


Since the program has multiple functions that are run on each request, try modifying the program to use only a single function per request. You can use a separate `handler` and route it to (for example) `/bench`. Combine this with `k6 run quickbench.js` to identify functions that run slowly.

Once some low-performing functions have been identified, write code to optimize them. Some modifications that can help performance:
* Use goroutines to process multiple things at a time
* Cache results of functions that run slowly
* Skip unneeded operations