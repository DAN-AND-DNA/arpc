# arpc

修改过的标准库rpc，暴露连接的信息，为了对接kcp和go-kit实现udp和接入微服务组件，也为了能获得连接的消息

## 基准
json为本包修改过的rpc，gob为官方的性能，但是数据量比protobuf还是太大了

```
=== RUN   Test_gob_rpc_call
--- PASS: Test_gob_rpc_call (0.00s)
=== RUN   Test_json_rpc_call
--- PASS: Test_json_rpc_call (0.00s)
goos: linux
goarch: amd64
pkg: template-project/benchmark
cpu: Intel(R) Core(TM) i5-9600K CPU @ 3.70GHz
Benchmark_server_handle_json_rpc_call
Benchmark_server_handle_json_rpc_call-6   	 1291899	       922.6 ns/op	    1960 B/op	      25 allocs/op
Benchmark_server_handle_gob_rpc_call
Benchmark_server_handle_gob_rpc_call-6    	  195685	      5492 ns/op	   16828 B/op	     220 allocs/op
PASS
```

## 测试

```
=== RUN   Test_client_json_rpc_call_easy
--- PASS: Test_client_json_rpc_call_easy (0.00s)
=== RUN   Test_client_json_rpc_call_map
--- PASS: Test_client_json_rpc_call_map (0.00s)
=== RUN   Test_client_json_rpc_call_bytes
--- PASS: Test_client_json_rpc_call_bytes (0.00s)
PASS
```


## example
```golang
func Benchmark_server_handle_json_rpc_call(b *testing.B) {
	b.ReportAllocs()

	b.RunParallel(func(pb *testing.PB) {
		dump_req := []byte(`{"method": "Dan.Ping", "id": "7", "params":["ping"]}`)

		conn := new(pkg_test_tool.Test_conn)

		server := pkg_rpc.New_server()
		d := &Dan{}
		if err := server.Register(d); err != nil {
			panic(err)
		}

		for pb.Next() {
			conn.Input_req(dump_req)
			server.Test_json(conn)
			conn.Reset()
		}
	})
}

func Benchmark_server_handle_gob_rpc_call(b *testing.B) {
	b.ReportAllocs()

	b.RunParallel(func(pb *testing.PB) {
		r := &arpc.Request{}
		r.Seq = 37
		r.ServiceMethod = "Dan.Ping"

		var enc_buf bytes.Buffer
		enc := gob.NewEncoder(&enc_buf)

		var args string = "ping"
		enc.Encode(r)
		enc.Encode(&args)

		dump_req := enc_buf.Bytes()

		conn := new(pkg_test_tool.Test_conn)
		server := pkg_rpc.New_server()
		d := &Dan{}
		if err := server.Register(d); err != nil {
			panic(err)
		}

		for pb.Next() {
			conn.Input_req(dump_req)
			server.Test(conn)
			conn.Reset()
		}
	})
}

```