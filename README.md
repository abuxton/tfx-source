# TFX Source

This source integrates with Terraform server either Terraform Enterprise (TFX) or Terraform Cloud (TFC) hence (TFx), allowing Overmind to pull data about many types of Terraform resources, and the Terraform API endpoints.

## Sources

### Organization

Gathers information about TFx Organizations <https://developer.hashicorp.com/terraform/cloud-docs/api-docs/organizations>

```json

```

#### `Get`

Gets a specific Organization by name.

**Query format:** The name of the Organization

#### `Find`

Finds all Organizations

### Workspace

Get Workspace info <https://developer.hashicorp.com/terraform/cloud-docs/api-docs/workspaces>

``` json

```

#### `Get`

Gets a specific workspace by ID.

**Query format:** The ID of the instance e.g. `i-09b0c0768577775ef`

#### `Find`

Finds all workspaces in an organization.

## Config

All configuration options can be provided via the command line or as environment variables:

| Environment Variable | CLI Flag | Automatic | Description |
|----------------------|----------|-----------|-------------|
| `CONFIG`| `--config` | ✅ | Config file location. Can be used instead of the CLI or environment variables if needed |
| `LOG`| `--log` | ✅ | Set the log level. Valid values: panic, fatal, error, warn, info, debug, trace |
| `NATS_SERVERS`| `--nats-servers` | ✅ | A list of NATS servers to connect to |
| `NATS_NAME_PREFIX`| `--nats-name-prefix` | ✅ | A name label prefix. Sources should append a dot and their hostname .{hostname} to this, then set this is the NATS connection name which will be sent to the server on CONNECT to identify the client |
| `NATS_JWT` | `--nats-jwt` | ✅ | The JWT token that should be used to authenticate to NATS, provided in raw format e.g. `eyJ0eXAiOiJKV1Q{...}` |
| `NATS_NKEY_SEED` | `--nats-nkey-seed` | ✅ | The NKey seed which corresponds to the NATS JWT e.g. `SUAFK6QUC{...}` |
| `MAX_PARALLEL`| `--max-parallel` | ✅ | Max number of requests to run in parallel |
| `AUTO_CONFIG` | `--auto-config` | | Use the local Terraform config, the same as the Terraform Cli could use. This can be set up with `TFE TODO` |
| `TFE_TOKEN` | `--tfe-token` | | The ID of the access key to use |
| `TFE_HOSTNAME` | `--tfe-hostname` | | The AWS region that this source should operate in |
### `srcman` config

When running in srcman, all of the above parameters marked with a checkbox are provided automatically, any additional parameters must be provided under the `config` key. These key-value pairs will become files in the `/etc/srcman/config` directory within the container.

```yaml
apiVersion: srcman.example.com/v0
kind: Source
metadata:
  name: source-sample
spec:
  image: ghcr.io/overmindtech/aws-source:latest
  replicas: 2
  manager: manager-sample
  config:
    # Example config values (not real credentials, don't panic)
    source.yaml: |
      tfe-token: m78yo9Hvh7Mruw...
      tfe-hostname: app.terraform.io
```

### Health Check

The source hosts a health check on `:8080/healthz` which will return an error if NATS is not connected. An example Kubernetes readiness probe is:

```yaml
readinessProbe:
  httpGet:
    path: /healthz
    port: 8080
```

## Development

### Source Type Naming Convention

The naming convention for types is as follows:

`{api}-{described_thing}-{version (optional)}`

**API:** The name of the api as it appears when making requests and based upon <https://developer.hashicorp.com/terraform/cloud-docs/api-docs> e.g. if you make requests to `https://app.terraform.io/api/v2/organizations/hashicorp` then the API name would be `app`.

**Described Thing:** What is being described as derived from the name of the API action. For example if the action was `Organizations` then the described thing would be `Organization`. Note that plurals should be converted so singular.

**Version:** This is an optional parameter used when required. For example the TFC API has the concept of `v1` and `v2` .

Some full examples of this naming convention therefore are:

* app-organization
* app-workspace-v1

Check the [Terraform API docs](https://developer.hashicorp.com/terraform/cloud-docs/api-docs) in order to predict other names.

### Running Locally

The source CLI can be interacted with locally by running:

```shell
go run main.go --help
```

### Contributing

The notes on contributing, testing and packaging have been moved to [CONTRIBUTING.md](./CONTRIBUTING.MD) and/or [OVERMIND.MD](./OVERMIND.MD)

