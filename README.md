# OPfuscate

### *An OPcache fork built to obfuscate your PHP source code with AES-256 encryption*

## Requirements

1. macOS (arm64 or x64)
2. A [static PHP binary](./bin/macos/) tailored for your selected architecture

## About

**OPfuscate** is a fork of OPcache that intercepts reads and writes to OPcache's file cache—transparently decrypting and encrypting data on the fly.

When you use OPcache's file cache exclusively, you can precompile your entire codebase into opcodes, eliminating the need to retain human-readable PHP files. **OPfuscate** secures these at-rest *opcodes* with AES-256-GCM, providing tamper-resistant encryption.

Each static PHP binary includes an obfuscated AES-256 key, which can be used by default. You may also supply additional authenticated data (AAD) via the `php.ini` file as a hex-encoded SHA-256 value.

By specifying a project-specific "license key" as your AAD, each project retains its own unique encryption context—even though all projects share the same underlying AES-256 key.

Whilst **OPfuscate** can also be delivered as a `.so` shared object, compiling it statically into a PHP binary (with debug symbols stripped) prevents the installation of additional, unauthorized extensions, thereby mitigating the risk of opcode dumping.

## Usage

First, download or clone this repository. Then, set the path to the static PHP binary:

```
$ ARCH="arm64"
$ PHP_BIN="./bin/macos/${ARCH}/php-8.4.3"

$ chmod +x "${PHP_BIN}"
```

**Note**: To prevent these demo PHP binaries from being used in production, opcode execution will be disabled after the start of March 3, 2025.

### Without AAD

```
$ export FILE="./php/aad-without.php"

$ "${PHP_BIN}" \
  -r 'opcache_compile_file(getenv("FILE")); file_put_contents(getenv("FILE"), "");'

$ "${PHP_BIN}" "${FILE}"
```

### With AAD

```
$ export FILE="./php/aad-with.php"

$ AAD="$("${PHP_BIN}" -r 'echo bin2hex(random_bytes(32));')"

$ "${PHP_BIN}" \
  -d opcache.opfuscate_aad="${AAD}" \
  -r 'opcache_compile_file(getenv("FILE")); file_put_contents(getenv("FILE"), "");'

$ "${PHP_BIN}" -d opcache.opfuscate_aad="${AAD}" "${FILE}"
```

In both cases (with and without AAD), the `.php` files will be zeroed out and the adjacent, encrypted `.php.bin` files will be executed.

## Credits

* [Christopher Pearse](https://x.com/chrisvpearse)
