# InfluxdbV2 Database Plugin HTTP API

The InfluxdbV2 database plugin is a plugin for the database
secrets engine. This plugin generates database credentials dynamically based on
configured roles for the Influxdb database.

## Build
To build this package for any platform you will need to clone this repository and cd into the repo directory and `go build -o inflxudbv2-database-plugin cmd/inflxudbv2-database-plugin/main.go`.

## Configure Connection

In addition to the parameters defined by the [Database
Secrets Engine](/api/secret/databases#configure-connection), this plugin
has a number of parameters to further configure a connection.

| Method | Path                     |
| :----- | :----------------------- |
| `POST` | `/database/config/:name` |

### Parameters

- `host` `(string: <required>)` – Specifies a Influxdb
  host to connect to.

- `port` `(int: 8086)` – Specifies the default port to use if none is provided
  as part of the host URI. Defaults to Influxdb's default transport port, 8086.

- `token` `(string: <required>)` – Specifies the API Token to use for
  superuser access.

- `tls` `(bool: true)` – Specifies whether to use TLS when connecting to
  Influxdb.

- `insecure_tls` `(bool: false)` – Specifies whether to skip verification of the
  server certificate when using TLS.

- `pem_bundle` `(string: "")` – Specifies concatenated PEM blocks containing a
  certificate and private key; a certificate, private key, and issuing CA
  certificate; or just a CA certificate.

- `pem_json` `(string: "")` – Specifies JSON containing a certificate and
  private key; a certificate, private key, and issuing CA certificate; or just a
  CA certificate. For convenience format is the same as the output of the
  `issue` command from the `pki` secrets engine; see
  [the pki documentation](/docs/secrets/pki).

- `connect_timeout` `(string: "5s")` – Specifies the connection timeout to use.

- `username_template` `(string)` - [Template](/docs/concepts/username-templating) describing how
  dynamic usernames are generated.

TLS works as follows:

- If `tls` is set to true, the connection will use TLS; this happens
  automatically if `pem_bundle`, `pem_json`, or `insecure_tls` is set

- If `insecure_tls` is set to true, the connection will not perform verification
  of the server certificate; this also sets `tls` to true

- If only `issuing_ca` is set in `pem_json`, or the only certificate in
  `pem_bundle` is a CA certificate, the given CA certificate will be used for
  server certificate verification; otherwise the system CA certificates will be
  used

- If `certificate` and `private_key` are set in `pem_bundle` or `pem_json`,
  client auth will be turned on for the connection

`pem_bundle` should be a PEM-concatenated bundle of a private key + client
certificate, an issuing CA certificate, or both. `pem_json` should contain the
same information; for convenience, the JSON format is the same as that output by
the issue command from the PKI secrets engine.

### Sample Payload

```json
{
  "plugin_name": "influxdbv2-database-plugin",
  "host": "influxdb1.local",
  "token": "asldkjklasjdlkadj.alshwdghjlkgjlkdfgjlkdsg==",
  "organization": "vault",
  "default_bucket": "the_leaky_bucket"
}
```

### Sample Request

```shell-session
$ curl \
    --header "X-Vault-Token: ..." \
    --request POST \
    --data @payload.json \
    http://127.0.0.1:8200/v1/influxdbv2/config/connection
```
