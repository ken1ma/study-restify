# curl で確認

デフォルトのレスポンスの `Content-Type` は JSON

    $ curl http://localhost:8080/hello/mark \
            --include --silent
    HTTP/1.1 200 OK
    Server: restify
    Content-Type: application/json
    Content-Length: 12
    Date: Sat, 13 Jan 2024 07:11:43 GMT
    Connection: keep-alive
    Keep-Alive: timeout=5

    "hello mark"

リクエストの `Accept` ヘッダーで、レスポンスの `Content-Type` を変更できる

    $ curl http://localhost:8080/hello/mark --header 'Accept: text/plain' \
            --include --silent
    HTTP/1.1 200 OK
    Server: restify
    Content-Type: text/plain
    Content-Length: 10
    Date: Sat, 13 Jan 2024 07:10:37 GMT
    Connection: keep-alive
    Keep-Alive: timeout=5

    hello mark

`HEAD` リクエストへの応答。

    $ curl http://localhost:8080/hello/mark --request HEAD --header 'Connection: close' \
            --include --silent
    HTTP/1.1 200 OK
    Server: restify
    Date: Sat, 13 Jan 2024 07:16:45 GMT
    Connection: close

`Connection: close` を指定しないと 5 秒待たされてしまう。

    $ time curl http://localhost:8080/hello/mark --request HEAD \
            --verbose --silent
    *   Trying 127.0.0.1:8080...
    * Connected to localhost (127.0.0.1) port 8080 (#0)
    > HEAD /hello/mark HTTP/1.1
    > Host: localhost:8080
    > User-Agent: curl/7.88.1
    > Accept: */*
    > 
    < HTTP/1.1 200 OK
    < Server: restify
    < Date: Sat, 13 Jan 2024 07:21:09 GMT
    < Connection: keep-alive
    < Keep-Alive: timeout=5
    * no chunk, no close, no size. Assume close to signal end
    < 
    * Closing connection 0

    real	0m5.011s
    user	0m0.002s
    sys	0m0.004s


# 実行時の警告

起動時に以下の警告が出力される

    [DEP0111] DeprecationWarning: Access to process.binding('http_parser') is deprecated.

1. [restify/node-restify#1876](https://github.com/restify/node-restify/issues/1876): restify が spdy を呼び出しているところで起きてる
    1. [spdy-http2/node-spdy#380](https://github.com/spdy-http2/node-spdy/issues/380): http-deceiver を呼び出しているところで起きてる
    1. [spdy-http2/http-deceiver#6](https://github.com/spdy-http2/http-deceiver/issues/6): `_http_common` に置き換えるべき
1. [restify/node-restify#1943](https://github.com/restify/node-restify/pull/1943): spdy が使われないときはロードしない PR が[マージ](https://github.com/restify/node-restify/commit/89e7ac81a4cc885d153df6f07d5cf35ed75fd4d0)されてるが [v11.1.0](https://github.com/restify/node-restify/releases/tag/v11.1.0) に含まれてない
