# shadowsocks


### **Build from source**

Use cargo to build. NOTE: **RAM >= 2GiB**

```bash
cargo build --release
```
## Supported Ciphers

1. Stream Ciphers Enabled (by default)

- `plain` or `none` (No encryption, only used for debugging or with plugins that ensure transport security)
- `table`
- `aes-128-cfb`, `aes-128-cfb1`, `aes-128-cfb8`, `aes-128-cfb128`
- `aes-192-cfb`, `aes-192-cfb1`, `aes-192-cfb8`, `aes-192-cfb128`
- `aes-256-cfb`, `aes-256-cfb1`, `aes-256-cfb8`, `aes-256-cfb128`
- `aes-128-ctr`, `aes-192-ctr`, `aes-256-ctr`
- `camellia-128-cfb`, `camellia-128-cfb1`, `camellia-128-cfb8`, `camellia-128-cfb128`
- `camellia-192-cfb`, `camellia-192-cfb1`, `camellia-192-cfb8`, `camellia-192-cfb128`
- `camellia-256-cfb`, `camellia-256-cfb1`, `camellia-256-cfb8`, `camellia-256-cfb128`
- `rc4-md5`
- `chacha20-ietf`

2. AEAD 2022 Ciphers
- `2022-blake3-aes-128-gcm`, `2022-blake3-aes-256-gcm`
- `2022-blake3-chacha20-poly1305`, `2022-blake3-chacha8-poly1305`

These Ciphers require `"password"` to be a Base64 string of key that have **exactly the same length** of Cipher's Key Size. It is recommended to use `ssservice genkey -m "METHOD_NAME"` to generate a secured and safe key.

3. AEAD Ciphers
- `chacha20-ietf-poly1305`
- `aes-128-gcm`, `aes-256-gcm`


Create a ShadowSocks' configuration file. Example

```jsonc
{
    "server": "my_server_ip",
    "server_port": 8388,
    "password": "rwQc8qPXVsRpGx3uW+Y3Lj4Y42yF9Bs0xg1pmx8/+bo=",
    "method": "aes-256-gcm",
    // ONLY FOR `sslocal`
    // Delete these lines if you are running `ssserver` or `ssmanager`
    "local_address": "127.0.0.1",
    "local_port": 1080
}
```

Start Shadowsocks client and server with:

```bash
sslocal -c config.json
ssserver -c config.json
```
## For configs and more details; see official shadowsocks-Rust [readme](https://github.com/shadowsocks/shadowsocks-rust/blob/master/README.md).



## GUI Shadowsocks-Rust
You can use [Shadowsocks-GUI-Rust](https://github.com/samanajd2/shadowsocks-gui-rust) Instead.
It is simple to use.


## License

[The MIT License (MIT)](https://opensource.org/licenses/MIT)

Copyright (c) 2014 Y. T. CHUNG

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.

## Stargazers over time

[![Stargazers over time](https://starchart.cc/shadowsocks/shadowsocks-rust.svg)](https://starchart.cc/shadowsocks/shadowsocks-rust)
