Outline:

1. Upstart cloud providers are making hosted javascript runtimes. These runtimes are providing super lightweight execution models. Lots of opportunity to explore new patterns that aren't available to bulky docker containers.
2. None of this stuff is open source, race to the bottom.
3. The future (could be) javascript and webassembly.
4. We'd need to implement core patterns in order to get this working:
   1. Js runtime
   2. Wasm runtime (separate or bundled)
   3. System calls and network/filesystem primitives.
   4. Distributed locks
   5. Slicer routing/transactional isolation
   6. Datastores. Global, eventually consistent KV store. SQL, OLAP.
   7. HTTPS certs, DNS, load balancers, etc etc etc
5. Fin?


How it will work:

- Implement slicer in Go
   - https://www.usenix.org/system/files/conference/osdi16/osdi16-adya.pdf
   - There is a lot of complexity to cover here, we'd need to handle most of it to get this to a really resilient state.
   - Slicer also manages traditional long-running jobs, need to think about how that changes with these quick jobs.
   - We get various transactional storage mechanisms out of slicer, so good to see if we can get a global KV store, or other storage types just out of slicer.
- Implement single-node js runner in Rust
   - Use v8 as runtime
      - Need to implement all built-ins not included in core V8
         - Console.log
         - Websocket
         - fetch
         - WASM is already all good
         - ???? many more, presumably
      - v8 bindings in rust: https://crates.io/crates/v8
         - Mem limits: https://docs.rs/v8/0.37.0/v8/struct.CreateParams.html#method.heap_limits
         - CPU limits: https://community.cloudflare.com/t/how-is-cpu-time-per-request-measured-in-cloudflare-workers/49964/3?u=maxm
         - Possible option for snapshotting: https://docs.rs/v8/0.37.0/v8/struct.SnapshotCreator.html
         - See various script caching options: https://docs.rs/v8/0.37.0/v8/script_compiler/index.html
         - Likely want to run isolates in their own process unless they're owned by the same author.
      - Serialization of js types: https://crates.io/crates/serde_v8
      - Example of minimal implementation: https://crates.io/crates/jstime_core
      - Watch this space? https://crates.io/crates/v8/reverse_dependencies
   - HTTP Server: https://github.com/hyperium/hyper
   - Need to run WASM
     - Already supported in V8 out of the box
     - Can just let people do it themselves, would only need to figure out shipping the wasm binary. Seems nice to allow first-class support though, where you can skip the JS piece if you want. If we do that then we need WASI.
     - Use wasi and wit/witx, generate bindings for necessary langs
       - https://github.com/jedisct1/witx-codegen
       - https://github.com/WebAssembly/WASI/blob/main/docs/witx.md
       - https://github.com/bytecodealliance/wit-bindgen
   - Need to provide a transactional kv store on the node
     - Probably just a mutex? Slicer can handle the rest


Deploy journey:
   1. Build wasm and/or js locally, spit out compatible artifacts.
   2. Need basic configuration:
      1. Memory limit, cpu time limit
      2. What to run: wasm function, class, function handler, etc...
      3. Load balancer/path rules?
      4. ??? more?
   3. Bundle is uploaded to registry
   4. Request is routed to node, if node doesn't have runtime it holds the request and downloads the bundle.
   5. Fin.

HTTP Request journey:
   1. TODO
   2. Need to support streaming body, large requests, request buffering, retries, lots of weird things here, but maybe write this when it's clearer what the separation is between slicer and the rust runtime.



Slicer Notes:
 - https://engineering.fb.com/2020/08/24/production-engineering/scaling-services-with-shard-manager/
 -
