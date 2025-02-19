# TLS Configuration Settings

Crypto TLS exposes a [variety of settings](https://godoc.org/crypto/tls).
Several of these settings are available for configuration within individual
receivers or exporters.

Note that mutual TLS (mTLS) is also supported.

## TLS / mTLS Configuration

By default, TLS is enabled:

- `insecure` (default = false): whether to enable client transport security for
  the exporter's gRPC connection. See
  [grpc.WithInsecure()](https://godoc.org/google.golang.org/grpc#WithInsecure).

As a result, the following parameters are also required:

- `cert_file`: Path to the TLS cert to use for TLS required connections. Should
  only be used if `insecure` is set to false.
- `key_file`: Path to the TLS key to use for TLS required connections. Should
  only be used if `insecure` is set to false.

A certificate authority may also need to be defined:

- `ca_file`: Path to the CA cert. For a client this verifies the server
  certificate. For a server this verifies client certificates. If empty uses
  system root CA. Should only be used if `insecure` is set to false.

Additionally you can configure TLS to be enabled but skip verifying the server's
certificate chain. This cannot be combined with `insecure` since `insecure`
won't use TLS at all.

- `insecure_skip_verify` (default = false): whether to skip verifying the
  certificate or not.

Minimum and maximum TLS version can be set:

__IMPORTANT__: TLS 1.0 and 1.1 are deprecated due to known vulnerabilities and should be avoided.

- `min_version` (default = "1.2"): Minimum acceptable TLS version.
  - options: ["1.0", "1.1", "1.2", "1.3"]

- `max_version` (default = "" handled by [crypto/tls](https://github.com/golang/go/blob/master/src/crypto/tls/common.go#L700) - currently TLS 1.3): Maximum acceptable TLS version.
  - options: ["1.0", "1.1", "1.2", "1.3"]

Additionally certificates may be reloaded by setting the below configuration.

- `reload_interval` (optional) : ReloadInterval specifies the duration after which the certificate will be reloaded.
   If not set, it will never be reloaded.
   Accepts a [duration string](https://pkg.go.dev/time#ParseDuration),
   valid time units are "ns", "us" (or "µs"), "ms", "s", "m", "h".

How TLS/mTLS is configured depends on whether configuring the client or server.
See below for examples.

## Client Configuration

[Exporters](https://github.com/open-telemetry/opentelemetry-collector/blob/main/exporter/README.md)
leverage client configuration. The TLS configuration parameters are defined
under `tls`, like server configuration.

Beyond TLS configuration, the following setting can optionally be configured:

- `server_name_override`: If set to a non-empty string, it will override the
  virtual host name of authority (e.g. :authority header field) in requests
  (typically used for testing).

Example:

```yaml
exporters:
  otlp:
    endpoint: myserver.local:55690
    tls:
      insecure: false
      ca_file: server.crt
      cert_file: client.crt
      key_file: client.key
      min_version: "1.1"
      max_version: "1.2"
  otlp/insecure:
    endpoint: myserver.local:55690
    tls:
      insecure: true
  otlp/secure_no_verify:
    endpoint: myserver.local:55690
    tls:
      insecure: false
      insecure_skip_verify: true
```

## Server Configuration

[Receivers](https://github.com/open-telemetry/opentelemetry-collector/blob/main/receiver/README.md)
leverage server configuration.

Beyond TLS configuration, the following setting can optionally be configured
(required for mTLS):

- `client_ca_file`: Path to the TLS cert to use by the server to verify a
  client certificate. (optional) This sets the ClientCAs and ClientAuth to
  RequireAndVerifyClientCert in the TLSConfig. Please refer to
  https://godoc.org/crypto/tls#Config for more information.

Example:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: mysite.local:55690
        tls:
          cert_file: server.crt
          key_file: server.key
  otlp/mtls:
    protocols:
      grpc:
        endpoint: mysite.local:55690
        tls:
          client_ca_file: client.pem
          cert_file: server.crt
          key_file: server.key
  otlp/notls:
    protocols:
      grpc:
        endpoint: mysite.local:55690
```
