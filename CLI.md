# Command Line Interface (CLI)

This covers more advanced CLI use cases and workflows. To [get started](README.md#getting-started) with the CLI, reference
the [README](README.md). Basic use of the CLI such as [creating a rule](CONTRIBUTING.md#creating-a-rule-with-the-cli) or
[testing](CONTRIBUTING.md#testing-a-rule-with-the-cli) are referenced in the [contribution guide](CONTRIBUTING.md).


## Using a config file or environment variables

CLI commands which are tied to Kibana and Elasticsearch are capable of parsing auth-related keyword args from a config
file or environment variables.

If a value is set in multiple places, such as config file and environment variable, the order of precedence will be as
follows:
* explicitly passed args (such as `--user joe`)
* environment variables
* config values
* prompt (this only applies to certain values)

#### Setup a config file

In the root directory of this repo, create the file `.detection-rules-cfg.json` and add relevant values

Currently supported arguments:
* elasticsearch_url
* kibana_url
* cloud_id
* *_username (kibana and es)
* *_password (kibana and es)

#### Using environment variables

Environment variables using the argument format: `DR_<UPPERCASED_ARG_NAME>` will be parsed in commands which expect it.
EX: `DR_USER=joe`


Using the environment variable `DR_BYPASS_NOTE_VALIDATION_AND_PARSE` will bypass the Detection Rules validation on the `note` field in toml files.

Using the environment variable `DR_BYPASS_BBR_LOOKBACK_VALIDATION` will bypass the Detection Rules lookback and interval validation
on the building block rules.

Using the environment variable `DR_BYPASS_TAGS_VALIDATION` will bypass the Detection Rules Unit Tests on the `tags` field in toml files.

## Importing rules into the repo

You can import rules into the repo using the `create-rule` or `import-rules` commands. Both of these commands will
require that the rules are schema-compliant and able to pass full validation. The biggest benefit to using these
commands is that they will strip[*](#note) additional fields[**](#note-2) and prompt for missing required
fields.

Alternatively, you can manually place rule files in the directory and run tests to validate as well.

<a id="note">\* Note</a>: This is currently limited to flat fields and may not apply to nested values.<br>
<a id="note-2">\** Note</a>: Additional fields are based on the current schema at the time the command is used.


#### `create-rule`

```console
Usage: detection_rules create-rule [OPTIONS] PATH

  Create a detection rule.

Options:
  -c, --config FILE               Rule or config file
  --required-only                 Only prompt for required fields
  -t, --rule-type [machine_learning|saved_query|query|threshold]
                                  Type of rule to create
  -h, --help                      Show this message and exit.
```

This command will allow you to pass a rule file using the `-c/--config` parameter. This is limited to one rule at a time
and will accept any valid rule in the following formats:
* toml
* json
* yaml (yup)
* ndjson (as long as it contains only a single rule and has the extension `.ndjson` or `.jsonl`)

#### `import-rules`

```console
Usage: detection_rules import-rules [OPTIONS] [INPUT_FILE]...

  Import rules from json, toml, or Kibana exported rule file(s).

Options:
  -d, --directory DIRECTORY  Load files from a directory
  -h, --help                 Show this message and exit.
```

The primary advantage of using this command is the ability to import multiple rules at once. Multiple rule paths can be
specified explicitly with unlimited arguments, recursively within a directory using `-d/--directory`[*](#note-3), or
a combination of both.

In addition to the formats mentioned using `create-rule`, this will also accept an `.ndjson`/`jsonl` file
containing multiple rules (as would be the case with a bulk export).

This will also strip additional fields and prompt for missing required fields.

<a id="note-3">\* Note</a>: This will attempt to parse ALL files recursively within a specified directory.


## Commands using Elasticsearch and Kibana clients

Commands which connect to Elasticsearch or Kibana are embedded under the subcommands:
* es
* kibana

These command groups will leverage their respective clients and will automatically use parsed config options if
defined, otherwise arguments should be passed to the sub-command as:

`python -m detection-rules kibana -u <username> -p <password> upload-rule <...>`


```console
python -m detection_rules es -h

Usage: detection_rules es [OPTIONS] COMMAND [ARGS]...

  Commands for integrating with Elasticsearch.

Options:
  -et, --timeout INTEGER        Timeout for elasticsearch client
  -ep, --es-password TEXT
  -eu, --es-user TEXT
  --cloud-id TEXT
  -e, --elasticsearch-url TEXT
  -h, --help                    Show this message and exit.

Commands:
  collect-events  Collect events from Elasticsearch.
```

Providers are the name that Elastic Cloud uses to configure authentication in Kibana. When we create deployment, Elastic Cloud configures two providers by default: basic/cloud-basic and saml/cloud-saml (for SSO).

```console
python -m detection_rules kibana -h

█▀▀▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄   ▄      █▀▀▄ ▄  ▄ ▄   ▄▄▄ ▄▄▄
█  █ █▄▄  █  █▄▄ █    █   █  █ █ █▀▄ █      █▄▄▀ █  █ █   █▄▄ █▄▄
█▄▄▀ █▄▄  █  █▄▄ █▄▄  █  ▄█▄ █▄█ █ ▀▄█      █ ▀▄ █▄▄█ █▄▄ █▄▄ ▄▄█

Usage: detection_rules kibana [OPTIONS] COMMAND [ARGS]...

  Commands for integrating with Kibana.

Options:
  --ignore-ssl-errors TEXT
  --space TEXT                 Kibana space
  --provider-name TEXT         For cloud deployments, Elastic Cloud configures
                               two providers by default: cloud-basic and
                               cloud-saml (for SSO)
  --provider-type TEXT         For cloud deployments, Elastic Cloud configures
                               two providers by default: basic and saml (for
                               SSO)
  -ku, --kibana-user TEXT
  --kibana-url TEXT
  -kp, --kibana-password TEXT
  -kc, --kibana-cookie TEXT    Cookie from an authed session
  --cloud-id TEXT              ID of the cloud instance. Defaults the cloud
                               provider to cloud-basic if this option is
                               supplied
  -h, --help                   Show this message and exit.

Commands:
  search-alerts  Search detection engine alerts with KQL.
  upload-rule    Upload a list of rule .toml files to Kibana.
```

## Searching Kibana for Alerts

Alerts stored in Kibana can be quickly be identified by searching with the `search-alerts` command.


```console
python -m detection_rules kibana search-alerts -h

Kibana client:
Options:
  --ignore-ssl-errors TEXT
  --space TEXT                 Kibana space
  --provider-name TEXT         For cloud deployments, Elastic Cloud configures
                               two providers by default: cloud-basic and
                               cloud-saml (for SSO)
  --provider-type TEXT         For cloud deployments, Elastic Cloud configures
                               two providers by default: basic and saml (for
                               SSO)
  -ku, --kibana-user TEXT
  --kibana-url TEXT
  -kp, --kibana-password TEXT
  -kc, --kibana-cookie TEXT    Cookie from an authed session
  --cloud-id TEXT              ID of the cloud instance. Defaults the cloud
                               provider to cloud-basic if this option is
                               supplied

Usage: detection_rules kibana search-alerts [OPTIONS] [QUERY]

  Search detection engine alerts with KQL.

Options:
  -d, --date-range <TEXT TEXT>...
                                  Date range to scope search
  -c, --columns TEXT              Columns to display in table
  -e, --extend                    If columns are specified, extend the
                                  original columns
  -h, --help                      Show this message and exit.
```

Running the following command will print out a table showing any alerts that have been generated recently.
`python3 -m detection_rules kibana --provider-name cloud-basic --kibana-url <url> --kibana-user <username> --kibana-password <password> search-alerts`

```console

█▀▀▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄   ▄      █▀▀▄ ▄  ▄ ▄   ▄▄▄ ▄▄▄
█  █ █▄▄  █  █▄▄ █    █   █  █ █ █▀▄ █      █▄▄▀ █  █ █   █▄▄ █▄▄
█▄▄▀ █▄▄  █  █▄▄ █▄▄  █  ▄█▄ █▄█ █ ▀▄█      █ ▀▄ █▄▄█ █▄▄ █▄▄ ▄▄█

===================================================================================================================================
 host                                 rule
 hostname                             name                                                                @timestamp
===================================================================================================================================
 stryker-malwares-MacBook-Pro.local   Sudo Heap-Based Buffer Overflow Attempt                             2022-06-21T14:08:34.288Z
 stryker-malwares-MacBook-Pro.local   Suspicious Automator Workflows Execution                            2022-06-21T13:58:30.857Z
 stryker-malwares-MacBook-Pro.local   Privilege Escalation Enumeration via LinPEAS                        2022-06-21T13:33:18.218Z
 stryker-malwares-MacBook-Pro.local   Privilege Escalation Enumeration via LinPEAS                        2022-06-21T13:28:14.685Z
 stryker-malwares-MacBook-Pro.local   Potential Reverse Shell Activity via Terminal                       2022-06-21T12:53:00.234Z
 stryker-malwares-MacBook-Pro.local   Potential Reverse Shell Activity via Terminal                       2022-06-21T12:53:00.237Z
 stryker-malwares-MacBook-Pro.local   Potential Kerberos Attack via Bifrost                               2022-06-20T20:33:53.810Z
 stryker-malwares-MacBook-Pro.local   Potential Kerberos Attack via Bifrost                               2022-06-20T20:33:53.813Z
 stryker-malwares-MacBook-Pro.local   Potential Privilege Escalation via Root Crontab File Modification   2022-06-20T20:23:50.557Z
 stryker-malwares-MacBook-Pro.local   Download and Execution of JavaScript Payload                        2022-06-20T20:18:46.211Z
===================================================================================================================================

```
## Uploading rules to Kibana

Toml formatted rule files can be uploaded as custom rules using the `kibana upload-rule` command. To upload more than one
file, specify multiple files at a time as individual args. This command is meant to support uploading and testing of
rules and is not intended for production use in its current state.

```console
python -m detection_rules kibana upload-rule -h

Kibana client:
Options:
  --space TEXT                 Kibana space
  -kp, --kibana-password TEXT
  -ku, --kibana-user TEXT
  --cloud-id TEXT
  -k, --kibana-url TEXT

Usage: detection_rules kibana upload-rule [OPTIONS]

  Upload a list of rule .toml files to Kibana.

Options:
  -f, --rule-file FILE
  -d, --directory DIRECTORY  Recursively export rules from a directory
  -id, --rule-id TEXT
  -r, --replace-id           Replace rule IDs with new IDs before export
  -h, --help                 Show this message and exit.
(detection-rules-build) (base) ➜  detection-rules git:(rule-loader) ✗
```

Alternatively, rules can be exported into a consolidated ndjson file which can be imported in the Kibana security app
directly.

```console
Usage: detection_rules export-rules [OPTIONS]

  Export rule(s) into an importable ndjson file.

Options:
  -f, --rule-file FILE
  -d, --directory DIRECTORY       Recursively export rules from a directory
  -id, --rule-id TEXT
  -o, --outfile FILE              Name of file for exported rules
  -r, --replace-id                Replace rule IDs with new IDs before export
  --stack-version [7.8|7.9|7.10|7.11|7.12]
                                  Downgrade a rule version to be compatible
                                  with older instances of Kibana
  -s, --skip-unsupported          If `--stack-version` is passed, skip rule
                                  types which are unsupported (an error will
                                  be raised otherwise)
  -h, --help                      Show this message and exit.
```

_*To load a custom rule, the proper index must be setup first. The simplest way to do this is to click
the `Load prebuilt detection rules and timeline templates` button on the `detections` page in the Kibana security app._


## Converting between JSON and TOML

[Importing rules](#importing-rules-into-the-repo) will convert from any supported format to toml. Additionally, the
command `view-rule` will also allow you to view a converted rule without importing it by specifying the `--rule-format` flag.

To view a rule in JSON format, you can also use the `view-rule` command with the `--api-format` flag, which is the default.
(See the [note](#a-note-on-version-handling) on the JSON formatted rules and versioning)


## A note on version handling

The rule toml files exist slightly different than they do in their final state as a JSON file in Kibana. The files are
white space stripped, normalized, sorted, and indented, prior to their json conversion. Everything within the `metadata`
table is also stripped out, as this is meant to be used only in the context of this repository and not in Kibana..

Additionally, the `version` of the rule is added to the file prior to exporting it. This is done to restrict version bumps
to occur intentionally right before we create a release. Versions are auto-incremented based on detected changes in
rules. This is based on the hash of the rule in the following format:
* sorted json
* serialized
* b64 encoded
* sha256 hash

As a result, all cases where rules are shown or converted to JSON are not just simple conversions from TOML.

## Debugging

Most of the CLI errors will print a concise, user friendly error. To enable debug mode and see full error stacktraces,
you can define `"debug": true` in your config file, or run `python -m detection-rules -d <commands...>`.

Precedence goes to the flag over the config file, so if debug is enabled in your config and you run
`python -m detection-rules --no-debug`, debugging will be disabled.


## Using `transform` in rule toml

A transform is any data that will be incorporated into _existing_ rule fields at build time, from within the 
`TOMLRuleContents.to_dict` method. _How_ to process each transform should be defined within the `Transform` class as a
method specific to the transform type.

### CLI support for investigation guide plugins

This applies to osquery and insights for the moment but could expand in the future.

```
(venv38) ➜  detection-rules-fork git:(2597-validate-osquery-insights) python -m detection_rules dev transforms -h

█▀▀▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄   ▄      █▀▀▄ ▄  ▄ ▄   ▄▄▄ ▄▄▄
█  █ █▄▄  █  █▄▄ █    █   █  █ █ █▀▄ █      █▄▄▀ █  █ █   █▄▄ █▄▄
█▄▄▀ █▄▄  █  █▄▄ █▄▄  █  ▄█▄ █▄█ █ ▀▄█      █ ▀▄ █▄▄█ █▄▄ █▄▄ ▄▄█

Usage: detection_rules dev transforms [OPTIONS] COMMAND [ARGS]...

  Commands for managing TOML [transform].

Options:
  -h, --help  Show this message and exit.

Commands:
  guide-plugin-convert  Convert investigation guide plugin format to toml
  guide-plugin-to-rule  Convert investigation guide plugin format to toml
```

`guide-plugin-convert` will print out the formatted toml.


```
(venv38) ➜  detection-rules-fork git:(2597-validate-osquery-insights) python -m detection_rules dev transforms guide-plugin-convert

█▀▀▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄▄▄ ▄   ▄      █▀▀▄ ▄  ▄ ▄   ▄▄▄ ▄▄▄
█  █ █▄▄  █  █▄▄ █    █   █  █ █ █▀▄ █      █▄▄▀ █  █ █   █▄▄ █▄▄
█▄▄▀ █▄▄  █  █▄▄ █▄▄  █  ▄█▄ █▄█ █ ▀▄█      █ ▀▄ █▄▄█ █▄▄ █▄▄ ▄▄█

Enter plugin contents []: !{osquery{"query":"SELECT description, display_name, name, path, pid, service_type, start_type, status, user_account FROM services\nWHERE NOT (user_account LIKE \"%LocalSystem\" OR user_account LIKE \"%LocalService\" OR user_account LIKE \"%NetworkService\" OR user_account == null)","label":"label2","ecs_mapping":{"labels":{"field":"description"},"agent.build.original":{"value":"fast"}}}}
[transform]

[[transform.osquery]]
query = "SELECT description, display_name, name, path, pid, service_type, start_type, status, user_account FROM services\nWHERE NOT (user_account LIKE \"%LocalSystem\" OR user_account LIKE \"%LocalService\" OR user_account LIKE \"%NetworkService\" OR user_account == null)"
label = "label2"

[transform.osquery.ecs_mapping]

[transform.osquery.ecs_mapping.labels]
field = "description"

[transform.osquery.ecs_mapping."agent.build.original"]
value = "fast"
```

The easiest way to _update_ a rule with existing transform entries is to use `guide-plugin-convert` and manually add it
to the rule.