# Synthetics command

Run Synthetics tests from your CI.

## Usage

### Setup

You need to either have `DATADOG_API_KEY` and `DATADOG_APP_KEY` in your environment or pass them to the CLI.
```bash
# Environment setup
export DATADOG_API_KEY="<API KEY>"
export DATADOG_APP_KEY="<APPLICATION KEY>"

# Passing to CLI
datadog-ci synthetics <command> --apiKey "<API KEY>" --appKey "<APPLICATION KEY>"
```

It is possible to configure the tool to use Datadog EU by defining the `DATADOG_SITE` environment variable to `datadoghq.eu`. By defaut the requests are sent to Datadog US.

If the org uses a custom sub-domain to access Datadog app, it needs to be set in the `DATADOG_SUBDOMAIN` environment variable to properly display the test results URL. As an example, if the URL used to access Datadog is `myorg.datadoghq.com` then set the environment variable to `myorg`, ie:

```bash
export DATADOG_SUBDOMAIN="myorg"
```

### API

By default it runs at the root of the working directory and finds `{,!(node_modules)/**/}*.synthetics.json` files (every files ending with `.synthetics.json` except those in the `node_modules` folder).

#### Configuration

Configuration is done via a json file, by default the tool load `datadog-ci.json` which can be overriden through the `--config` argument.

The configuration file structure is the following:

```json
{
    "apiKey": "<DATADOG_API_KEY>",
    "appKey": "<DATADOG_APPLICATION_KEY>",
    "datadogSite": "datadoghq.com",
    "files": "{,!(node_modules)/**/}*.synthetics.json",
    "global": {
        "allowInsecureCertificates": true,
        "basicAuth": { "username": "test", "password": "test" },
        "body": "{\"fakeContent\":true}",
        "bodyType": "application/json",
        "cookies": "name1=value1;name2=value2;",
        "deviceIds": ["laptop_large"],
        "followRedirects": true,
        "headers": { "NEW_HEADER": "NEW VALUE" },
        "locations": ["aws:us-east-1"],
        "retry": { "count": 2, "interval": 300 },
        "skip": true,
        "startUrl": "{{URL}}?static_hash={{STATIC_HASH}}",
        "variables": { "titleVariable": "new title" },
    },
    "timeout": 120000,
}
```

#### Commands

The available command is:

- `run-tests`: run the tests discovered in the folder according to the `files` configuration key

It accepts the `--public-id` (or shorthand `-p`) argument to trigger only the specified test. It can be set multiple times to run multiple tests:

```bash
datadog-ci synthetics run-test --public-id pub-lic-id1 --public-id pub-lic-id2
```

### Test files

Your test files must be named with a `.synthetics.json` suffix.

```json
// myTest.synthetics.json
{
    "tests": [
        {
            "id": "<TEST_PUBLIC_ID>",
            "config": {
                "allowInsecureCertificates": true,
                "basicAuth": { "username": "test", "password": "test" },
                "body": "{\"fakeContent\":true}",
                "bodyType": "application/json",
                "cookies": "name1=value1;name2=value2;",
                "deviceIds": ["laptop_large"],
                "followRedirects": true,
                "headers": { "NEW_HEADER": "NEW VALUE" },
                "locations": ["aws:us-east-1"],
                "retry": { "count": 2, "interval": 300 },
                "skip": true,
                "startUrl": "{{URL}}?static_hash={{STATIC_HASH}}",
                "variables": { "titleVariable": "new title" },
            }
        }
    ]
}
```

The `<TEST_PUBLIC_ID>` can be either the identifier of the test found in the URL of a test details page (eg. for `https://app.datadoghq.com/synthetics/details/abc-def-ghi` it would be `abc-def-ghi`) or the full URL to the details page (ie. directly `https://app.datadoghq.com/synthetics/details/abc-def-ghi`).

You can configure on which url your test starts by providing a `config.startUrl` to your test object and build your own starting url using any part of your test's original starting url and the following environment variables: 

| Environment variable | Description                  | Example                                                |
|----------------------|------------------------------|--------------------------------------------------------|
| `URL`                | Test's original starting url | `https://www.example.org:81/path/to/something?abc=123` |
| `DOMAIN`             | Test's domain name           | `example.org`                                          |
| `HOST`               | Test's host                  | `www.example.org:81`                                   |
| `HOSTNAME`           | Test's hostname              | `www.example.org`                                      |
| `ORIGIN`             | Test's origin                | `https://www.example.org:81`                           |
| `PARAMS`             | Test's query parameters      | `?abc=123`                                             |
| `PATHNAME`           | Test's URl path              | `/path/to/something`                                   |
| `PORT`               | Test's host port             | `81`                                                   |
| `PROTOCOL`           | Test's protocol              | `https:`                                               |
| `SUBDOMAIN`          | Test's sub domain            | `www`                                                  |

For instance, if your test's starting url is `https://www.example.org:81/path/to/something?abc=123`

It can be written as :

* `{{PROTOCOL}}//{{SUBDOMAIN}}.{{DOMAIN}}:{{PORT}}{{PATHNAME}}{{PARAMS}}`
* `{{PROTOCOL}}//{{HOST}}{{PATHNAME}}{{PARAMS}}`
* `{{URL}}`

and so on...