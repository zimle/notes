# Zstandard

Is a [compression format](https://github.com/facebook/zstd) developed by Yann Collet at Facebook, see [wiki](https://en.wikipedia.org/wiki/Zstd).

Usage

```shell
# compress all files in folder 299 to a file 299.tar.zst
tar --zstd -cf output.tar.zst 299
# uncompress 299.tar.zst
tar --zstd -xf 299.tar.zst
```
