slt is a dead-simple TLS reverse-proxy with SNI multiplexing (TLS virtual hosts).

That means you can send TLS/SSL connections for multiple different applications to the same port and forward
them all to the appropriate backend hosts depending on the intended destination.

# Features

### SNI Multiplexing
slt multiplexes connections to a single TLS port by inspecting the name in the SNI extension field of each connection.

### Simple YAML Configuration
You configure slt with a simple YAML configuration file:

    bind_addr: ":443"

    frontends:
      v1.example.com:
        backends:
          -
            addr: ":4443"

      v2.example.com:
        backends:
          -
            addr: "192.168.0.2:443"
          -
            addr: "192.168.0.1:443"


### Optional Default Frontend
Connections that do not match other frontends will use the default frontend. Only one frontend may be the default.

```yaml
frontends:
  v1.example.com:
    backends:
      -
        addr: ":4443"
  example.com:
    default: true
    backends:
      -
        addr: ":443"
```

### Optional TLS Termination
Sometimes, you don't actually want to terminate the TLS traffic, you just want to forward it elsewhere. slt only
terminates the TLS traffic if you specify a private key and certificate file like so:

    frontends:
      v1.example.com:
        tls_key: /path/to/v1.example.com.key
        tls_crt: /path/to/v1.example.com.crt


### TLS Backend with Optinal Signature Verification and SNI Removal

```yaml
    frontends:
      v3.example.com:
        backends:
          -
            addr: "tls://foo.example.com:8443"
            signature: ""                             # signature will be printed but not verified
          -
            addr: "tls://foo.example.com:8443"
            signature: "verify"                       # signature will be verified using built-in verification
          -
            addr: "tls://foo.example.com:8443"
            signature: "xxx"                          # signature must match "xxx"
          -
            addr: "tls://tcp://bar.example.com:8443"  # connection to bar.example.com won't contain SNI
            signature: "verify"                       # signature will be verified against "bar.example.com"
```

### Round robin load balancing among arbitrary backends
slt performs simple round-robin load balancing when more than one backend is available (other strategies will be available in the future):

    frontends:
      v1.example.com:
        backends:
          -
            addr: ":8080"
          -
            addr: ":8081"


# Running it
Running slt is also simple. It takes a single argument, the path to the configuration file:

    ./slt /path/to/config.yml


# Building it
Just cd into the directory and "go build". It requires Go 1.1+.

# Testing it
Just cd into the directory and "go test".

# Stability
I run slt in production handling hundreds of thousands of connections daily.

# License
Apache
