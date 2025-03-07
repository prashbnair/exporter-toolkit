# Web configuration

Exporters and services instrumented with the Exporter Toolkit share the same
web configuration file format. This is *experimental* and might change in the
future.

To specify which web configuration file to load, use the `--web.config.file` flag.

The file is written in [YAML format](https://en.wikipedia.org/wiki/YAML),
defined by the scheme described below.
Brackets indicate that a parameter is optional. For non-list parameters the
value is set to the specified default.

The file is read upon every http request, such as any change in the
configuration, so the certificates are picked up immediately.

Generic placeholders are defined as follows:

* `<boolean>`: a boolean that can take the values `true` or `false`
* `<filename>`: a valid path in the current working directory
* `<secret>`: a regular string that is a secret, such as a password
* `<string>`: a regular string

```
tls_server_config:
  # Certificate and key files for server to use to authenticate to client.
  cert_file: <filename>
  key_file: <filename>

  # Server policy for client authentication. Maps to ClientAuth Policies.
  # For more detail on clientAuth options:
  # https://golang.org/pkg/crypto/tls/#ClientAuthType
  #
  # NOTE: If you want to enable client authentication, you need to use
  # RequireAndVerifyClientCert. Other values are insecure.
  [ client_auth_type: <string> | default = "NoClientCert" ]

  # CA certificate for client certificate authentication to the server.
  [ client_ca_file: <filename> ]

  # Minimum TLS version that is acceptable.
  [ min_version: <string> | default = "TLS12" ]

  # Maximum TLS version that is acceptable.
  [ max_version: <string> | default = "TLS13" ]

  # List of supported cipher suites for TLS versions up to TLS 1.2. If empty,
  # Go default cipher suites are used. Available cipher suites are documented
  # in the go documentation:
  # https://golang.org/pkg/crypto/tls/#pkg-constants
  [ cipher_suites:
    [ - <string> ] ]

  # prefer_server_cipher_suites controls whether the server selects the
  # client's most preferred ciphersuite, or the server's most preferred
  # ciphersuite. If true then the server's preference, as expressed in
  # the order of elements in cipher_suites, is used.
  [ prefer_server_cipher_suites: <bool> | default = true ]

  # Elliptic curves that will be used in an ECDHE handshake, in preference
  # order. Available curves are documented in the go documentation:
  # https://golang.org/pkg/crypto/tls/#CurveID
  [ curve_preferences:
    [ - <string> ] ]

http_server_config:
  # Enable HTTP/2 support. Note that HTTP/2 is only supported with TLS.
  # This can not be changed on the fly.
  [ http2: <boolean> | default = true ]

# Usernames and hashed passwords that have full access to the web
# server via basic authentication. If empty, no basic authentication is
# required. Passwords are hashed with bcrypt.
basic_auth_users:
  [ <string>: <secret> ... ]
```

[A sample configuration file](web.yml) is provided.

## About bcrypt

There are several tools out there to generate bcrypt passwords, e.g.
[htpasswd](https://httpd.apache.org/docs/2.4/programs/htpasswd.html):

`htpasswd -nBC 10 "" | tr -d ':\n'`

That command will prompt you for a password and output the hashed password,
which will look something like:
`$2y$10$X0h1gDsPszWURQaxFh.zoubFi6DXncSjhoQNJgRrnGs7EsimhC7zG`

The cost (10 in the example) influences the time it takes for computing the
hash. A higher cost will end up slowing down the authentication process.
Depending on the machine, a cost of 10 will take about ~70ms, whereas a cost of
18 can take up to a few seconds. That hash will be computed on the first
authenticated HTTP request and then cached.

## Performance

Basic authentication is meant for simple use cases, with a few users.  If you
need to authenticate a lot of users, it is recommended to use TLS client
certificates, or to use a proper reverse proxy to handle the authentication.
