# shadowsocks


### **Build from source**

Use cargo to build. NOTE: **RAM >= 2GiB**

```bash
cargo build --release
```

Then `sslocal` and `ssserver` will appear in `./target/(debug|release)/`, it works similarly as the two binaries in the official ShadowSocks' implementation.

```bash
make install TARGET=release
```

Then `sslocal`, `ssserver`, `ssmanager` and `ssurl` will be installed to `/usr/local/bin` (variable PREFIX).

For Windows users, if you have encountered any problem in building, check and discuss in [#102](https://github.com/shadowsocks/shadowsocks-rust/issues/102).

```

## Getting Started

Generate a safe and secured password for a specific encryption method (`aes-128-gcm` in the example) with:

```bash
ssservice genkey -m "aes-128-gcm"
```

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

Detailed explanation could be found in [shadowsocks' documentation](https://github.com/shadowsocks/shadowsocks/wiki).

In shadowsocks-rust, we also have an extended configuration file format, which is able to define more than one server. You can also disable individual servers.

```jsonc
{
    "servers": [
        {
            "server": "127.0.0.1",
            "server_port": 8388,
            "password": "rwQc8qPXVsRpGx3uW+Y3Lj4Y42yF9Bs0xg1pmx8/+bo=",
            "method": "aes-256-gcm",
            "timeout": 7200
        },
        {
            "server": "127.0.0.1",
            "server_port": 8389,
            "password": "/dliNXn5V4jg6vBW4MnC1I8Jljg9x7vSihmk6UZpRBM=",
            "method": "chacha20-ietf-poly1305"
        },
        {
            "disabled": true,
            "server": "eg.disable.me",
            "server_port": 8390,
            "password": "mGvbWWay8ueP9IHnV5F1uWGN2BRToiVCAWJmWOTLU24=",
            "method": "chacha20-ietf-poly1305"
        }
    ],
    // ONLY FOR `sslocal`
    // Delete these lines if you are running `ssserver` or `ssmanager`
    "local_port": 1080,
    "local_address": "127.0.0.1"
}
```

`sslocal` automatically selects the best server with the lowest latency and the highest availability.

Start Shadowsocks client and server with:

```bash
sslocal -c config.json
ssserver -c config.json
```

## Usage

Start local client with configuration file

```bash
# Read local client configuration from file
sslocal -c /path/to/shadowsocks.json
```

### Socks5 Local client

```bash
# Pass all parameters via command line
sslocal -b "127.0.0.1:1080" -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty" --plugin "v2ray-plugin" --plugin-opts "server;tls;host=github.com"

# Pass server with SIP002 URL
sslocal -b "127.0.0.1:1080" --server-url "ss://YWVzLTI1Ni1nY206cGFzc3dvcmQ@127.0.0.1:8388/?plugin=v2ray-plugin%3Bserver%3Btls%3Bhost%3Dgithub.com"
```

### HTTP Local client

```bash
sslocal -b "127.0.0.1:3128" --protocol http -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty"
```

All parameters are the same as Socks5 client, except `--protocol http`.

### Tunnel Local client

```bash
# Set 127.0.0.1:8080 as the target for forwarding to
sslocal --protocol tunnel -b "127.0.0.1:3128" -f "127.0.0.1:8080" -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty"
```

- `--protocol tunnel` enables local client Tunnel mode
- `-f "127.0.0.1:8080` sets the tunnel target address

### Transparent Proxy Local client

**NOTE**: It currently only supports

- Linux (with `iptables` targets `REDIRECT` and `TPROXY`)
- BSDs (with `pf`), such as OS X 10.10+, FreeBSD, ...

```bash
sslocal -b "127.0.0.1:60080" --protocol redir -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty" --tcp-redir "redirect" --udp-redir "tproxy"
```

Redirects connections with `iptables` configurations to the port that `sslocal` is listening on.

- `--protocol redir` enables local client Redir mode
- (optional) `--tcp-redir` sets TCP mode to `REDIRECT` (Linux)
- (optional) `--udp-redir` sets UDP mode to `TPROXY` (Linux)

### Tun interface client

**NOTE**: It currently only supports

- Linux, Android
- macOS, iOS

#### Linux

Create a Tun interface with name `tun0`

```bash
ip tuntap add mode tun tun0
ifconfig tun0 inet 10.255.0.1 netmask 255.255.255.0 up
```

Start `sslocal` with `--protocol tun` and binds to `tun0`

```bash
sslocal --protocol tun -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty" --outbound-bind-interface lo0 --tun-interface-name tun0
```

#### macOS

```bash
sslocal --protocol tun -s "[::1]:8388" -m "aes-256-gcm" -k "hello-kitty" --outbound-bind-interface lo0 --tun-interface-address 10.255.0.1/24
```

It will create a Tun interface with address `10.255.0.1` and netmask `255.255.255.0`.

### Server

```bash
# Read server configuration from file
ssserver -c /path/to/shadowsocks.json

# Pass all parameters via command line
ssserver -s "[::]:8388" -m "aes-256-gcm" -k "hello-kitty" --plugin "v2ray-plugin" --plugin-opts "server;tls;host=github.com"
```

### Server Manager

Supported [Manage Multiple Users](https://github.com/shadowsocks/shadowsocks/wiki/Manage-Multiple-Users) API:

- `add` - Starts a server instance
- `remove` - Deletes an existing server instance
- `list` - Lists all current running servers
- `ping` - Lists all servers' statistic data

NOTE: `stat` command is not supported. Because servers are running in the same process with the manager itself.

```bash
# Start it just with --manager-address command line parameter
ssmanager --manager-address "127.0.0.1:6100"

# For *nix system, manager can bind to unix socket address
ssmanager --manager-address "/tmp/shadowsocks-manager.sock"

# You can also provide a configuration file
#
# `manager_address` key must be provided in the configuration file
ssmanager -c /path/to/shadowsocks.json

# Create one server by UDP
echo 'add: {"server_port":8388,"password":"hello-kitty"}' | nc -u '127.0.0.1' '6100'

# Close one server by unix socket
echo 'remove: {"server_port":8388}' | nc -Uu '/tmp/shadowsocks-manager.sock'
```

For manager UI, check more details in the [shadowsocks-manager](https://github.com/shadowsocks/shadowsocks-manager) project.

Example configuration:

```jsonc
{
    // Required option
    // Address that ssmanager is listening on
    "manager_address": "127.0.0.1",
    "manager_port": 6100,

    // Or bind to a Unix Domain Socket
    "manager_address": "/tmp/shadowsocks-manager.sock",

    "servers": [
        // These servers will be started automatically when ssmanager is started
    ],

    // Outbound socket binds to this IP address
    // For choosing different network interface on the same machine
    "local_address": "xxx.xxx.xxx.xxx",

    // Other options that may be passed directly to new servers
}
```

## Configuration

```jsonc
{
    // LOCAL: Listen address. This is exactly the same as `locals[0]`
    // SERVER: Bind address for remote sockets, mostly used for choosing interface
    //         Don't set it if you don't know what's this for.
    "local_address": "127.0.0.1",
    "local_port": 1080,

    // Extended multiple local configuration
    "locals": [
        {
            // Basic configuration, a SOCKS5 local server
            "local_address": "127.0.0.1",
            "local_port": 1080,
            // OPTIONAL. Setting the `mode` for this specific local server instance.
            // If not set, it will derive from the outer `mode`
            "mode": "tcp_and_udp",
            // OPTIONAL. Authentication configuration file
            // Configuration file document could be found in the next section.
            "socks5_auth_config_path": "/path/to/auth.json",
            // OPTIONAL. Instance specific ACL
            "acl": "/path/to/acl/file.acl",
        },
        {
            // SOCKS5, SOCKS4/4a local server
            "protocol": "socks",
            // Listen address
            "local_address": "127.0.0.1",
            "local_port": 1081,
            // OPTIONAL. Enables UDP relay
            "mode": "tcp_and_udp",
            // OPTIONAL. Customizing the UDP's binding address. Depending on `mode`, if
            // - TCP is enabled, then SOCKS5's UDP Association command will return this address
            // - UDP is enabled, then SOCKS5's UDP server will listen to this address.
            "local_udp_address": "127.0.0.1",
            "local_udp_port": 2081
        },
        {
            // Tunnel local server (feature = "local-tunnel")
            "protocol": "tunnel",
            // Listen address
            "local_address": "127.0.0.1",
            "local_port": 5353,
            // Forward address, the target of this tunnel
            // In this example, this will build a `127.0.0.1:5353` -> `8.8.8.8:53` tunnel
            "forward_address": "8.8.8.8",
            "forward_port": 53,
            // OPTIONAL. Customizing whether to start TCP and UDP tunnel
            "mode": "tcp_only"
        },
        {
            // HTTP local server (feature = "local-http")
            "protocol": "http",
            // Listen address
            "local_address": "127.0.0.1",
            "local_port": 3128
        },
        {
            // DNS local server (feature = "local-dns")
            // This DNS works like China-DNS, it will send requests to `local_dns` and `remote_dns` and choose by ACL rules
            "protocol": "dns",
            // Listen address
            "local_address": "127.0.0.1",
            "local_port": 53,
            // OPTIONAL. DNS local server uses `tcp_and_udp` mode by default
            "mode": "udp_only",
            // Local DNS address, DNS queries will be sent directly to this address
            "local_dns_address": "114.114.114.114",
            // OPTIONAL. Local DNS's port, 53 by default
            "local_dns_port": 53,
            // Remote DNS address, DNS queries will be sent through ssserver to this address
            "remote_dns_address": "8.8.8.8",
            // OPTIONAL. Remote DNS's port, 53 by default
            "remote_dns_port": 53
        },
        {
            // Tun local server (feature = "local-tun")
            "protocol": "tun",
            // Tun interface name
            "tun_interface_name": "tun0",
            // Tun interface address
            //
            // It has to be a host address in CIDR form
            "tun_interface_address": "10.255.0.1/24"
        },
        {
            // Transparent Proxy (redir) local server (feature = "local-redir")
            "protocol": "redir",
            // OPTIONAL: TCP type, may be different between platforms
            // Linux/Android: redirect (default), tproxy
            // FreeBSD/OpenBSD: pf (default), ipfw
            // NetBSD/macOS/Solaris: pf (default), ipfw
            "tcp_redir": "tproxy",
            // OPTIONAL: UDP type, may be different between platforms
            // Linux/Android: tproxy (default)
            // FreeBSD/OpenBSD: pf (default)
            "udp_redir": "tproxy"
        }
    ],

    // Server configuration
    // listen on :: for dual stack support, no need add [] around.
    "server": "::",
    // Change to use your custom port number
    "server_port": 8388,
    "method": "aes-256-gcm",
    "password": "your-password",
    "plugin": "v2ray-plugin",
    "plugin_opts": "mode=quic;host=github.com",
    "plugin_args": [
        // Each line is an argument passed to "plugin"
        "--verbose"
    ],
    "plugin_mode": "tcp_and_udp", // SIP003u, default is "tcp_only"
    // Server: TCP socket timeout in seconds.
    // Client: TCP connection timeout in seconds.
    // Omit this field if you don't have specific needs.
    "timeout": 7200,

    // Extended multiple server configuration
    // LOCAL: Choosing the best server to connect dynamically
    // SERVER: Creating multiple servers in one process
    "servers": [
        {
            // Fields are the same as the single server's configuration

            // Individual servers can be disabled
            // "disabled": true,
            "address": "0.0.0.0",
            "port": 8389,
            "method": "aes-256-gcm",
            "password": "your-password",
            "plugin": "...",
            "plugin_opts": "...",
            "plugin_args": [],
            "plugin_mode": "...",
            "timeout": 7200,

            // Customized weight for local server's balancer
            //
            // Weight must be in [0, 1], default is 1.0.
            // The higher weight, the server may rank higher.
            "tcp_weight": 1.0,
            "udp_weight": 1.0,

            // OPTIONAL. Instance specific ACL
            "acl": "/path/to/acl/file.acl",
        },
        {
            // Same key as basic format "server" and "server_port"
            "server": "0.0.0.0",
            "server_port": 8388,
            "method": "chacha20-ietf-poly1305",
            // Read the actual password from environment variable PASSWORD_FROM_ENV
            "password": "${PASSWORD_FROM_ENV}"
        },
        {
            // AEAD-2022
            "server": "::",
            "server_port": 8390,
            "method": "2022-blake3-aes-256-gcm",
            "password": "3SYJ/f8nmVuzKvKglykRQDSgg10e/ADilkdRWrrY9HU=",
            // For Server (OPTIONAL)
            // Support multiple users with Extensible Identity Header
            // https://github.com/Shadowsocks-NET/shadowsocks-specs/blob/main/2022-2-shadowsocks-2022-extensible-identity-headers.md
            "users": [
                {
                    "name": "username",
                    // User's password must have the same length as server's password
                    "password": "4w0GKJ9U3Ox7CIXGU4A3LDQAqP6qrp/tUi/ilpOR9p4="
                }
            ],
            // For Client (OPTIONAL)
            // If EIH enabled, then "password" should have the following format: iPSK:iPSK:iPSK:uPSK
            // - iPSK is one of the middle relay servers' PSK, for the last `ssserver`, it must be server's PSK ("password")
            // - uPSK is the user's PSK ("password")
            // Example:
            // "password": "3SYJ/f8nmVuzKvKglykRQDSgg10e/ADilkdRWrrY9HU=:4w0GKJ9U3Ox7CIXGU4A3LDQAqP6qrp/tUi/ilpOR9p4="
        }
    ],

    // Global configurations for UDP associations
    "udp_timeout": 300, // Timeout for UDP associations (in seconds), 5 minutes by default
    "udp_max_associations": 512, // Maximum UDP associations to be kept in one server, unlimited by default

    // Options for Manager
    "manager_address": "127.0.0.1", // Could be a path to UNIX socket, /tmp/shadowsocks-manager.sock
    "manager_port": 5300, // Not needed for UNIX socket

    // DNS server's address for resolving domain names
    // For *NIX and Windows, it uses system's configuration by default
    //
    // Value could be IP address of DNS server, for example, "8.8.8.8".
    // DNS client will automatically request port 53 with both TCP and UDP protocol.
    //
    // - system, uses system provided API (`getaddrinfo` on *NIX)
    //
    // It also allows some pre-defined well-known public DNS servers:
    // - google (TCP, UDP)
    // - cloudflare (TCP, UDP)
    // - cloudflare_tls (TLS), enable by feature "dns-over-tls"
    // - cloudflare_https (HTTPS), enable by feature "dns-over-https"
    // - quad9 (TCP, UDP)
    // - quad9_tls (TLS), enable by feature "dns-over-tls"
    //
    // The field is only effective if feature "trust-dns" is enabled.
    "dns": "google",
    // Configure `cache_size` for "trust-dns" ResolverOpts. Set to "0" to disable DNS cache.
    "dns_cache_size": 0,

    // Mode, could be one of the
    // - tcp_only
    // - tcp_and_udp
    // - udp_only
    "mode": "tcp_only",

    // TCP_NODELAY
    "no_delay": false,

    // Enables `SO_KEEPALIVE` and set `TCP_KEEPIDLE`, `TCP_KEEPINTVL` to the specified seconds
    "keep_alive": 15,

    // Soft and Hard limit of file descriptors on *NIX systems
    "nofile": 10240,

    // Try to resolve domain name to IPv6 (AAAA) addresses first
    "ipv6_first": false,
    // Set IPV6_V6ONLY for all IPv6 listener sockets
    // Only valid for locals and servers listening on `::`
    "ipv6_only": false,

    // Outbound socket options
    // Linux Only (SO_MARK)
    "outbound_fwmark": 255,
    // FreeBSD only (SO_USER_COOKIE)
    "outbound_user_cookie": 255,
    // `SO_BINDTODEVICE` (Linux), `IP_BOUND_IF` (BSD), `IP_UNICAST_IF` (Windows) socket option for outbound sockets
    "outbound_bind_interface": "eth1",
    // Outbound socket bind() to this IP (choose a specific interface)
    "outbound_bind_addr": "11.22.33.44",

    // Balancer customization
    "balancer": {
        // MAX Round-Trip-Time (RTT) of servers
        // The timeout seconds of each individual checks
        "max_server_rtt": 5,
        // Interval seconds between each check
        "check_interval": 10,
        // Interval seconds between each check for the best server
        // Optional. Specify to enable shorter checking interval for the best server only.
        "check_best_interval": 5
    },

    // Service configurations
    // Logger configuration
    "log": {
        // Equivalent to `-v` command line option
        "level": 1,
        "format": {
            // Euiqvalent to `--log-without-time`
            "without_time": false,
        },
        // Equivalent to `--log-config`
        // More detail could be found in https://crates.io/crates/log4rs
        "config_path": "/path/to/log4rs/config.yaml"
    },
    // Runtime configuration
    "runtime": {
        // single_thread or multi_thread
        "mode": "multi_thread",
        // Worker threads that are used in multi-thread runtime
        "worker_count": 10
    }
}
```

### SOCKS5 Authentication Configuration

The configuration file is set by `socks5_auth_config_path` in `locals`.

```jsonc
{
    // Password/Username Authentication (RFC1929)
    "password": {
        "users": [
            {
                "user_name": "USERNAME in UTF-8",
                "password": "PASSWORD in UTF-8"
            }
        ]
    }
}
```

### Environment Variables

- `SS_SERVER_PASSWORD`: A default password for servers that created from command line argument (`--server-addr`)
- `SS_SYSTEM_DNS_RESOLVER_FORCE_BUILTIN`: `"system"` DNS resolver force use system's builtin (`getaddrinfo` in *NIX)

## Supported Ciphers

### AEAD 2022 Ciphers

- `2022-blake3-aes-128-gcm`, `2022-blake3-aes-256-gcm`
- `2022-blake3-chacha20-poly1305`, `2022-blake3-chacha8-poly1305`

These Ciphers require `"password"` to be a Base64 string of key that have **exactly the same length** of Cipher's Key Size. It is recommended to use `ssservice genkey -m "METHOD_NAME"` to generate a secured and safe key.

### AEAD Ciphers

- `chacha20-ietf-poly1305`
- `aes-128-gcm`, `aes-256-gcm`

### Stream Ciphers

- `plain` or `none` (No encryption, only used for debugging or with plugins that ensure transport security)

- `table`
- `aes-128-cfb`, `aes-128-cfb1`, `aes-128-cfb8`, `aes-128-cfb128`
- `aes-192-cfb`, `aes-192-cfb1`, `aes-192-cfb8`, `aes-192-cfb128`
- `aes-256-cfb`, `aes-256-cfb1`, `aes-256-cfb8`, `aes-256-cfb128`
- `aes-128-ctr`
- `aes-192-ctr`
- `aes-256-ctr`
- `camellia-128-cfb`, `camellia-128-cfb1`, `camellia-128-cfb8`, `camellia-128-cfb128`
- `camellia-192-cfb`, `camellia-192-cfb1`, `camellia-192-cfb8`, `camellia-192-cfb128`
- `camellia-256-cfb`, `camellia-256-cfb1`, `camellia-256-cfb8`, `camellia-256-cfb128`
- `rc4-md5`
- `chacha20-ietf`


## ACL

`sslocal`, `ssserver`, and `ssmanager` support ACL file with syntax like [shadowsocks-libev](https://github.com/shadowsocks/shadowsocks-libev). Some examples could be found in [here](https://github.com/shadowsocks/shadowsocks-libev/tree/master/acl).

### Available sections

- For local servers (`sslocal`, `ssredir`, ...)
  - Modes:
    - `[bypass_all]` - ACL runs in `BlackList` mode. Bypasses all addresses that didn't match any rules.
    - `[proxy_all]` - ACL runs in `WhiteList` mode. Proxies all addresses that didn't match any rules.
  - Rules:
    - `[bypass_list]` - Rules for connecting directly
    - `[proxy_list]` - Rules for connecting through proxies
- For remote servers (`ssserver`)
  - Modes:
    - `[reject_all]` - ACL runs in `BlackList` mode. Rejects all clients that didn't match any rules.
    - `[accept_all]` - ACL runs in `WhiteList` mode. Accepts all clients that didn't match any rules.
  - Rules:
    - `[white_list]` - Rules for accepted clients
    - `[black_list]` - Rules for rejected clients
    - `[outbound_block_list]` - Rules for blocking outbound addresses.

### Example

```ini
# SERVERS
# For ssserver, accepts requests from all clients by default
[accept_all]

# Blocks these clients
[black_list]
1.2.3.4
127.0.0.1/8

# Disallow these outbound addresses
[outbound_block_list]
127.0.0.1/8
::1
# Using regular expression
^[a-z]{5}\.baidu\.com
# Match exactly
|baidu.com
# Match with subdomains
||google.com
# An internationalized domain name should be converted to punycode
# |☃-⌘.com - WRONG
|xn----dqo34k.com
# ||джpумлатест.bрфa - WRONG
||xn--p-8sbkgc5ag7bhce.xn--ba-lmcq

# CLIENTS
# For sslocal, ..., bypasses all targets by default
[bypass_all]

# Proxy these addresses
[proxy_list]
||google.com
8.8.8.8
```

## Useful Tools

1. `ssurl` is for encoding and decoding ShadowSocks URLs (SIP002). Example:

  ```plain
  ss://YWVzLTI1Ni1jZmI6cGFzc3dvcmQ@127.0.0.1:8388/?plugin=obfs-local%3Bobfs%3Dhttp%3Bobfs-host%3Dwww.baidu.com
  ```

## Notes

It supports the following features:

- [x] SOCKS5 CONNECT command
- [x] SOCKS5 UDP ASSOCIATE command (partial)
- [x] SOCKS4/4a CONNECT command
- [x] Various crypto algorithms
- [x] Load balancing (multiple servers) and server delay checking
- [x] [SIP004](https://github.com/shadowsocks/shadowsocks-org/issues/30) AEAD ciphers
- [x] [SIP003](https://github.com/shadowsocks/shadowsocks-org/issues/28) Plugins
- [x] [SIP003u](https://github.com/shadowsocks/shadowsocks-org/issues/180) Plugin with UDP support
- [x] [SIP002](https://github.com/shadowsocks/shadowsocks-org/issues/27) Extension ss URLs
- [x] [SIP022](https://github.com/shadowsocks/shadowsocks-org/issues/196) AEAD 2022 ciphers
- [x] HTTP Proxy Supports ([RFC 7230](http://tools.ietf.org/html/rfc7230) and [CONNECT](https://tools.ietf.org/html/draft-luotonen-web-proxy-tunneling-01))
- [x] Defend against replay attacks, [shadowsocks/shadowsocks-org#44](https://github.com/shadowsocks/shadowsocks-org/issues/44)
- [x] Manager APIs, supporting [Manage Multiple Users](https://github.com/shadowsocks/shadowsocks/wiki/Manage-Multiple-Users)
- [x] ACL (Access Control List)
- [x] Support HTTP/HTTPS Proxy protocol

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
