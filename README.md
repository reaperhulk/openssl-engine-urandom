## OpenSSL osrandom engine

This is a very simple engine that can replace the OpenSSL CSPRNG with /dev/urandom (and CryptGenRandom on Windows). Why would anyone want to do this? Well, you can avoid reseed after fork issues that many applications run into when consuming OpenSSL. Additionally, if you believe it's easier to make mistakes with seeding userland CSPRNGs then bypassing OpenSSL's in favor of the host OS makes sense.

###Caveats

If you prefer the OpenSSL CSPRNG over /dev/urandom then you should not use this. Otherwise the current version of this engine has the following issues:

* /dev/urandom is slower than the OpenSSL CSPRNG. If your application requires >10MB/sec of entropy look elsewhere.

/dev/urandom is not a panacea and the existence of this engine should not be considered evidence that it is superior to OpenSSL's CSPRNG.

#### Compiling

You'll need a compiler (clang/gcc) and OpenSSL.

```
clang -c -fpic e_osrandom.c
clang -shared -lcrypto -o e_osrandom.so
```

If you're compiling against an alternate OpenSSL (like one from homebrew) it might look more like this:

```
clang -c -fpic -I/usr/local/opt/openssl/include e_osrandom.c
clang -shared -L/usr/local/opt/openssl/lib e_osrandom.o -lcrypto -o e_osrandom.so
```

#### Using It

Load the engine and set it as the default RAND (in C):

```c
ENGINE *e = ENGINE_by_id("dynamic");
if (!ENGINE_ctrl_cmd_string(e, "SO_PATH", "/path/to/e_osrandom.so", 0)) {
  printf("error");
}
if (!ENGINE_ctrl_cmd_string(e, "ID", "osrandom", 0)) {
  printf("error");
}
if (!ENGINE_ctrl_cmd_string(e, "LOAD", NULL, 0)) {
  printf("error");
}
if (!ENGINE_set_default_RAND(e)) {
  printf("error");
}
```
