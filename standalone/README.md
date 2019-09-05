# Web Assembly Standalone
The commands to run this code.
```
emcc standalone.c -s EXPORTED_FUNCTIONS='["_doubler"]' -Os -s WASM=1 -s SIDE_MODULE=1 -o standalone.wasm

emrun --port 8080 .
```

## The Details

When compiling to WebAssembly, `emcc` emits both a JavaScript file and a WebAssembly file. The JavaScript loads the WebAssembly which contains the compiled code. This is necessary in many cases as WebAssembly currently depends on a JavaScript runtime for features like longjmp, C++ exceptions, checking the date or time, printing to the console, using an API like WebGL, etc. However, if your WebAssembly only contains **pure computational code**, it may require almost no JavaScript support. If it in fact doesn't need any of the Emscripten runtime, then it is completely "**standalone**", and all you need to use it is some [JavaScript to load it](http://webassembly.org/getting-started/js-api/). Alternatively, it may need some runtime but you may prefer to write your own runtime code, or perhaps your runtime is not even a JS VM (e.g. [wasmjit](https://github.com/rianhunter/wasmjit)).

### Emitting wasm by itself

Normally emcc will emit wasm and JS to load it. You can simply throw out the JS and load the wasm with your own code. It is better in that case, though, to tell emcc you will do that, so it can avoid optimizations that assume the JS and wasm are tightly coupled. To do that, just tell emcc to only emit a wasm file,
```
emcc source.c -o output.wasm
```

### JS/wasm ABI

emcc can optionally emit useful metadata in the wasm file, that a runtime can use. See the `EMIT_EMSCRIPTEN_METADATA` option, added in [#7815](https://github.com/kripken/emscripten/pull/7815). The metadata includes versioning as well as things like the memory and table sizes the wasm needs, that the runtime needs to provide.

### Let the optimizer remove the runtime

As of 1.37.29, `emcc`'s optimizer is powerful enough to remove all runtime elements that are not used (it does this using meta-dce, dead code elimination that crosses the JavaScript/WebAssembly boundary). This can be helpful as then the necessary runtime is smaller and easier to replace. To try this, simply build with something like

```
emcc source.c -Os
```

`-Os` is needed to make the optimizer work at full power (you can also use `-Oz` or `-O3`). The result will be two files, JavaScript and WebAssembly as usual. You can then inspect the WebAssembly file and see what it imports: if you are only doing pure computation, it should import little or nothing, in which case it is easy to write your own [loading code](http://webassembly.org/getting-started/js-api/) to use it.

 * If you use C++ objects, the compiler must in many cases emit code to catch exceptions so that RAII destructors are called, and exceptions require a lot of runtime support. Build with `-fno-exceptions` to disable exceptions entirely.
 * [More details on how this works](https://hacks.mozilla.org/2018/01/shrinking-webassembly-and-javascript-code-sizes-in-emscripten/).

### Create a dynamic library

Dynamic libraries have a formal definition, and are designed to be loadable in a standard way. That is a benefit over the previous approach, however, dynamic libraries also have downsides, such as having relocations for memory and function pointers, which add overhead that may be unnecessary if you are only using one module (and not linking several together).

To build a dynamic library, use
````
emcc source.c -s SIDE_MODULE=1 -o target.wasm
````

That will emit the output dynamic library as `target.wasm`.

To use a side module, see `loadWebAssemblyModule` in [`src/runtime.js`](https://github.com/kripken/emscripten/blob/65271b0ed8e77d07ced7f6873c3582b6b0ae2719/src/runtime.js#L351) for some example loading code.
 
 * The conventions for a wasm dynamic library are [here](https://github.com/WebAssembly/tool-conventions/blob/master/DynamicLinking.md).
 * Note that there is no special handling of a C stack. A module can have one internally if it wants one (it needs to ask for the memory for it, then handle it however it wants).
 * [Full example](https://gist.github.com/kripken/59c67556dc03bb6d57052fedef1e61ab).
 * The LLVM wasm backend does not support dynamic libraries yet, so you can only use this with asm2wasm (fastcomp).
