# W9Y

Musings on a webassembly k8s. This is all about determinism. Should be able to run things either in javascript or webassembly and trivially replay, re-run, previous executions. Blank slate architecture will allow for these kinds of opportunities and we should proactively grab them.

Need a way to run services. Need to define how they communicate. Cloudflare's approach here is nice, you just pack everything into `fetch()` calls, the url that fetch hits defines what service you're communicating with. How does this work for private requests?

Three (general) types of request patterns:
1. Respond to an action (like an http request or cron)
2. Long running, or really "transactionally bound" context. Doesn't actually need to be a transactional process, but needs to be able to make decisions like one. Think slicer, or temporal, or cloudflares poorly named "durable objects".

So we need a runner that can simulate requests to these types of runners, inject faults, and store a record of simulation runs for later debugging. Should have fuzzing, replay, pausing, time travel, etc...

The entire thing would be built around V8, runs javascript first and then WASI directly through the browser runtime.

## Hard problems

1. How are jobs configured and stored. Want to be declarative without becoming k8s spaghetti. Maybe start with terraform.
2. How are requests routed. Likely built-in or use something that already exists. Our usage patterns might permit something very simple though.
3. Service to service type safety, protbuf, wix, etc etc etc, would be nice to have this, or make it a pluggable thing. Type checking on service to service communication would be nice to at leas think about proactively supporting. Keep in mind this is all happening over fetch now. Can we map urls to typed schemas? Maybe just allow users to type protobuf/openapi/whatever and

## Resources

- Temporal: https://github.com/temporalio/temporal
- Replay.io
  - Driver specs: https://static.replay.io/driver/
  - Their chrome fork: https://github.com/RecordReplay/chromium
  - More: https://static.replay.io/protocol/
- V8 rust package: https://crates.io/crates/v8
  - Minimal example: https://github.com/jstime/jstime
- Popularity for various api schema specifications: https://www.postman.com/state-of-api/api-technologies/
-


