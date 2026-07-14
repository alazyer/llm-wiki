# How Modern Browsers Work

## Summary
Modern browsers are multi-process application runtimes that fetch resources, parse HTML/CSS/JavaScript, compute layout, paint display lists, composite GPU-backed layers, execute JavaScript, and isolate untrusted web content through sandboxing and site isolation.

## Key Takeaways
- A navigation starts in the browser process: URL parsing, security checks, DNS, TLS, HTTP request/response handling, redirects, caching, and renderer selection.
- Modern networking optimizes latency through HTTP/2 and HTTP/3 multiplexing, preconnect, DNS prefetch, Early Hints, preload scanners, Fetch Priority, and Speculation Rules.
- The renderer builds the DOM from streaming HTML, parses CSS into CSSOM/style data, and pauses on blocking scripts unless `async`, `defer`, or modules change execution behavior.
- Style calculation resolves the cascade and inheritance into computed styles; layout turns the visual DOM into geometry with sizes, positions, line boxes, flex/grid decisions, and dirty-region recalculation.
- Painting records drawing operations; compositing groups content into layers, rasterizes tiles, and lets the GPU assemble frames for scrolling and transform/opacity animations.
- V8 runs JavaScript through parsing, bytecode, Ignition, Sparkplug, Maglev, and TurboFan/Turboshaft tiers, while garbage collection manages young and old generations incrementally and concurrently.
- ES modules create dependency graphs that are fetched, parsed, instantiated, and evaluated in dependency order; dynamic `import()` and import maps extend this loading model.
- Chromium uses browser, renderer, GPU, network, utility, extension, and sometimes per-site iframe processes to improve stability, security, and performance isolation.
- Site isolation and sandboxing protect origins and the OS, especially after Spectre-class attacks, at the cost of additional process and memory overhead.
- Chromium, Gecko, and WebKit share the same broad pipeline but differ in style engines, compositor/raster strategies, JavaScript engines, process allocation, and tooling.

## Related
- [[generative-ui/generative-ui-new-frontend-patterns]]
- [[ai-assisted-software-engineering/agentic-code-review-verification-bottleneck]]
- [[llm-inference-systems/kv-caching-in-llms]]
