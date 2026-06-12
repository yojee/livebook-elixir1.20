# Livebook (Yojee fork)

This is **Yojee's fork of [Livebook](https://github.com/livebook-dev/livebook)**,
maintained for Yojee's internal use. It tracks upstream Livebook and publishes a
Docker image to Yojee's GitHub Container Registry.

For general Livebook documentation, features, and guides, see the
[upstream project](https://github.com/livebook-dev/livebook) and the
[official docs](https://hexdocs.pm/livebook/).

## Running with Docker

The Yojee image is published to `ghcr.io/yojee/yojee-livebook-elixir`.

```shell
# Run with the default configuration
docker run -p 8080:8080 -p 8081:8081 ghcr.io/yojee/yojee-livebook-elixir:latest

# Set a password (must be at least 12 characters)
docker run -p 8080:8080 -p 8081:8081 \
  -e LIVEBOOK_PASSWORD="securesecret" \
  ghcr.io/yojee/yojee-livebook-elixir:latest

# Mount a local directory to read/save notebooks on your machine.
# Run as your user so created files have the right permissions.
docker run -p 8080:8080 -p 8081:8081 \
  -u $(id -u):$(id -g) -v $(pwd):/data \
  ghcr.io/yojee/yojee-livebook-elixir:latest
```

Then open http://localhost:8080.

## Publishing a new image

The image is built and pushed by the [`Release`](.github/workflows/release.yml)
GitHub Actions workflow. Trigger it manually (`workflow_dispatch`) and provide an
image tag (e.g. `yojee-0.1.0`); optionally tag it as `latest`. The workflow builds
for `linux/amd64` and `linux/arm64`.

For the version number, bump based on the latest in ghcr.io/yojee/yojee-livebook-elixir,
e.g. if it's `yojee-0.1.0` and you made a minor fix, then bump to `yojee-0.1.1`. If you're
building for a new version of Elixir, then bump to `yojee-0.2.0`.

## Environment variables

The section below is also embedded into `livebook server --help` at compile time
(see `lib/livebook_cli/server.ex`), which reads this file and splits on the two
surrounding HTML comment markers — so do not remove them.

<!-- Environment variables -->

The following environment variables can be used to configure Livebook on boot:

  * `LIVEBOOK_ALLOW_URI_SCHEMES` - sets additional allowed hyperlink schemes to the
    Markdown content. Livebook sanitizes links in Markdown, allowing only a few
    standard schemes by default (such as http and https). Set it to a comma-separated
    list of schemes.

  * `LIVEBOOK_APP_SERVICE_NAME` - sets the application name used by the cloud
    provider to aid debugging.

  * `LIVEBOOK_APP_SERVICE_URL` - sets the application url to manage this
    Livebook instance within the cloud provider platform.

  * `LIVEBOOK_APPS_BANNER` - sets the value to render at the top apps banner.

  * `LIVEBOOK_APPS_PATH` - the directory with app notebooks. When set, the apps
    are deployed on Livebook startup with the persisted settings. Password-protected
    notebooks will receive a random password, unless `LIVEBOOK_APPS_PATH_PASSWORD`
    is set. When deploying using Livebook's Docker image, consider using
    `LIVEBOOK_APPS_PATH_WARMUP`.

  * `LIVEBOOK_APPS_PATH_PASSWORD` - the password to use for all protected apps
    deployed from `LIVEBOOK_APPS_PATH`.

  * `LIVEBOOK_APPS_PATH_WARMUP` - sets the warmup mode for apps deployed from
    `LIVEBOOK_APPS_PATH`. Must be either "auto" (apps are warmed up on Livebook
    startup, right before app deployment) or "manual" (apps are warmed up when
    building the Docker image; to do so add "RUN /app/bin/warmup_apps" to
    your image). Defaults to "auto".

  * `LIVEBOOK_AWS_CREDENTIALS` - enable Livebook to read AWS Credentials from
    environment variables, AWS Credentials, EC2/ECS metadata when configuring
    S3 buckets.

  * `LIVEBOOK_BASE_URL_PATH` - sets the base url path the web application is
    served on. Useful when deploying behind a reverse proxy.

  * `LIVEBOOK_PUBLIC_BASE_URL_PATH` - sets the base url path the `/public/*` routes
    are served on. Note that this takes precedence over `LIVEBOOK_BASE_URL_PATH`,
    if both are set. Setting this may be useful to create exceptions when deploying
    behind a reverse proxy that requires authentication.

  * `LIVEBOOK_CACERTFILE` - path to a local file containing CA certificates.
    Those certificates are used during for server authentication when Livebook
    accesses files from external sources.

  * `LIVEBOOK_CLUSTER` - configures clustering strategy when running multiple
    instances of Livebook using either the Docker image or an Elixir release.
    See the "Clustering" docs for more information:
    https://hexdocs.pm/livebook/clustering.html

  * `LIVEBOOK_COOKIE` - sets the cookie for running Livebook in a cluster.
    Defaults to a random string that is generated on boot.

  * `LIVEBOOK_DATA_PATH` - the directory to store Livebook's internal
    configuration. Defaults to "livebook" under the default user data
    directory.

  * `LIVEBOOK_DEFAULT_RUNTIME` - sets the runtime type that is used by default
    when none is started explicitly for the given notebook. Must be either
    "standalone" (Standalone), "attached:NODE:COOKIE" (Attached node)
    or "embedded" (Embedded). Defaults to "standalone".

  * `LIVEBOOK_FORCE_SSL_HOST` - sets a host to redirect to if the request is not over HTTPS.
    Note it does not apply when accessing Livebook via localhost. Defaults to nil.

  * `LIVEBOOK_HOME` - sets the home path for the Livebook instance. This is the
    default path used on file selection screens and others. Defaults to the
    user's operating system home.

  * `LIVEBOOK_IDENTITY_PROVIDER` - controls whether Zero Trust Authentication
    must be used for this Livebook instance. This is useful when deploying
    Livebook inside a cloud platform, such as Cloudflare and Google.
    Supported values are:

      * `basic_auth:<username>:<password>`
      * `cloudflare:<your-team-name (domain)>`
      * `google_iap:<your-audience (aud)>`
      * `tailscale:<tailscale-cli-socket-path>`
      * `custom:YourElixirModule`

    See our authentication docs for more information: https://hexdocs.pm/livebook/authentication.html

  * `LIVEBOOK_IFRAME_PORT` - sets the port that Livebook serves iframes at.
    This is relevant only when running Livebook without TLS. Defaults to 8081.

  * `LIVEBOOK_IFRAME_URL` - sets the URL that Livebook loads iframes from.
    By default iframes are loaded from local `LIVEBOOK_IFRAME_PORT` when accessing
    Livebook over http:// and from https://livebookusercontent.com when accessing over https://.

  * `LIVEBOOK_IMAGE_REGISTRY_URL` - sets the container image registry used to fetch livebook images from.
    By default uses `ghcr.io/livebook-dev/livebook`.

  * `LIVEBOOK_IP` - sets the ip address to start the web application on.
    Must be a valid IPv4 or IPv6 address.

  * `LIVEBOOK_LOG_LEVEL` - sets the logger level, allowing for more verbose
    logging, either of: error, warning, notice, info, debug. Defaults to warning.

  * `LIVEBOOK_LOG_METADATA` - a comma-separated list of metadata keys that should
    be included in the log messages. Livebook-specific keys include:
    - `users` (attached to evaluation and request logs)
    - `session_mode` (attached to evaluation logs, either "default" or "app")
    - `code` (attached to evaluation logs, the code being evaluated)
    - `event` (attached to evaluation logs, currently always "code.evaluate")

    By default includes only `request_id`.

  * `LIVEBOOK_LOG_FORMAT` - sets the log output format, either "text" (default)
    for human-readable logs or "json" for structured JSON.

  * `LIVEBOOK_NODE` - sets the node name for running Livebook in a cluster.
    Note that Livebook always runs using long names distribution, so the
    node host name must use a fully qualified domain name (FQDN) or an IP
    address.

  * `LIVEBOOK_PASSWORD` - sets a password that must be used to access Livebook.
    Must be at least 12 characters. Defaults to token authentication.

  * `LIVEBOOK_PROXY_HEADERS` - a comma-separated list of headers that are set by
    proxies. For example, `x-forwarded-for,x-forwarded-proto`. Configuring those
    may be required when running Livebook behind reverse proxies.

  * `LIVEBOOK_PORT` - sets the port Livebook runs on. If you want to run multiple
    instances on the same domain with the same credentials but on different ports,
    you also need to set `LIVEBOOK_SECRET_KEY_BASE`. Defaults to 8080. If set to 0,
    a random port will be picked.

  * `LIVEBOOK_SECRET_KEY_BASE` - sets a secret key that is used to sign and encrypt
    the session and other payloads used by Livebook. Must be at least 64 characters
    long and it can be generated by commands such as: `openssl rand -base64 48`.
    Defaults to a random secret on every boot.

  * `LIVEBOOK_SHUTDOWN_ENABLED` - controls if a shutdown button should be shown
    in the homepage. Set it to "true" to enable it.

  * `LIVEBOOK_TOKEN_ENABLED` - controls whether token authentication is enabled.
    Enabled by default unless `LIVEBOOK_PASSWORD` is set. Set it to "false" to
    disable it.

  * `LIVEBOOK_UPDATE_INSTRUCTIONS_URL` - sets the URL to direct the user to for
    updating Livebook when a new version becomes available.

  * `LIVEBOOK_WITHIN_IFRAME` - controls if the application is running inside an
    iframe. Set it to "true" to enable it. If you do enable it, then the application
    must run with HTTPS.

The environment variables `ERL_AFLAGS` and `ERL_ZFLAGS` can also be set to configure
Livebook and the notebook runtimes. `ELIXIR_ERL_OPTIONS` are also available to customize
Livebook, but it is not forwarded to runtimes.

<!-- Environment variables -->

## License

This project is a fork of Livebook and retains its original license.

Copyright (C) 2021 Dashbit

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
