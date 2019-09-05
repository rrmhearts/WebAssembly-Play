# Web Assembly
Repo to play with Web Assembly

## Developer's Guide Example
This is a hello world tutorial. Repo consists of final product.
```
$ git clone https://github.com/emscripten-core/emsdk.git
$ cd emsdk
$ ./emsdk install latest
$ ./emsdk activate latest
```

**OR** build yourself (my choice)
```
$ git clone https://github.com/emscripten-core/emsdk.git
$ cd emsdk
$ ./emsdk install --build=Release sdk-incoming-64bit binaryen-master-64bit
$ ./emsdk activate --build=Release sdk-incoming-64bit binaryen-master-64bit
```

When finished. Source environment.
`$ source ./emsdk_env.sh --build=Release`

Compile and run a simple program. **Found in** `hello/`
```
$ mkdir hello
$ cd hello
$ cat << EOF > hello.c
#include <stdio.h>
int main(int argc, char ** argv) {
  printf("Hello, world!\n");
}
EOF
$ emcc hello.c -o hello.html
```

Start a server.
` $ emrun --no_browser --port 8080 . `

The C code runs inside of a pre-fabricated website with a small console window in the middle. It generates the HTML and all of pre-fab site. Prints `"Hello, world!"` to console.

## Other Examples
* See SIMPLER Example in `hellojs/`
* See more complex example in `standalone/`

## How does it work

On machine compiles C code to WASM byte code which is understood by Web Browsers. It is a "second language" to JavaScript that has been implemented on all the major browsers. It acts as a Java bytecode which any language is compiled into.
