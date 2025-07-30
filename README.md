# nfrastack/container-clamav

## About

This repository will build a container with [Clam Antivirus](https://www.clamav.net) to scan files or mail messages.

* Auto Configuration Support
* Automatic Downlad and update of Virus Definitions
* Ability to load custom definitions
* Log rotation

## Maintainer

- [Nfrastack](https://www.nfrastack.com)

## Table of Contents

- [About](#about)
- [Maintainer](#maintainer)
- [Table of Contents](#table-of-contents)
- [Installation](#installation)
  - [Prebuilt Images](#prebuilt-images)
  - [Quick Start](#quick-start)
  - [Persistent Storage](#persistent-storage)
- [Configuration](#configuration)
  - [Environment Variables](#environment-variables)
    - [Base Images used](#base-images-used)
    - [Core Configuration](#core-configuration)
    - [Virus Definitions Configuration](#virus-definitions-configuration)
    - [Virus Scanning Settings](#virus-scanning-settings)
    - [Scanning Limits](#scanning-limits)
    - [Alerting Settings](#alerting-settings)
  - [Users and Groups](#users-and-groups)
  - [Networking](#networking)
- [Maintenance](#maintenance)
  - [Shell Access](#shell-access)
  - [Manual Definition Updates](#manual-definition-updates)
- [Support & Maintenance](#support--maintenance)
- [License](#license)

## Installation


### Prebuilt Images
Feature limited builds of the image are available on the [Github Container Registry](https://github.com/nfrastack/container-clamav/pkgs/container/container-clamav)
To unlock advanced features, one must provide a code to be able to change specific environment variables from defaults. Support the development to gain access to a code.

```
(container platform) pull ghcr.io/nfrastack/container-clamav:(image_tag)
```

Image tag syntax is:
<image>:<optional tag>-<distribution>_<distribution_release>_<optional_features>
Example: `ghcr.io/nfrastack-container-clamav:1.0-alpine_3.22_core` or
         `ghcr.io/nfrastack-container-clamav:alpine_3.22` or
         `ghcr.io/nfrastack-container-clamav:debian_trixie`

* An otpional tag may exist that matches the [CHANGELOG](CHANGELOG.md)
* Distribution can either be `alpine` or `debian`
* Disribution release varies - see the registry for availability
* Features
  * `core` - Includes `age`, `cron`, `logrotate`, `s6overlay`, `msmtp`
  * `faflopza` - Includes everything in core and `fail2ban`, `fluentbit`, `openbao`, `zabbix` - *This is the default image if no features requested*
  * `faflopzaze` Includes everything in `faflopza` and `zerotier`.

#### Multi-Architecture Support

Images are built for `amd64` by default, with optional support for `arm64` and other architectures.

### Quick Start

* The quickest way to get started is using [docker-compose](https://docs.docker.com/compose/). See the examples folder for a working [compose.yml](examples/compose.yml) that can be modified for your use.

* Map [persistent storage](#data-volumes) for access to configuration and data files for backup.
* Set various [environment variables](#environment-variables) to understand the capabilities of this image.

### Persistent Storage

The following directories are used for configuration and can be mapped for persistent storage.

| Directory           | Description         |
| ------------------- | ------------------- |
| `/config`      | Configuration Files |
| `/data` | AntiVirus Definitions   |
| `/logs`             | Log Files           |

### Environment Variables

#### Base Images used

This image relies on a customized base image in order to work.
Be sure to view the following repositories to understand all the customizable options:

| Image                                                   | Description |
| ------------------------------------------------------- | ----------- |
| [OS Base](https://github.com/nfrastack/container-base/) | Base Image  |

Below is the complete list of available options that can be used to customize your installation.

* Variables showing an 'x' under the `Advanced` column can only be set if the containers advanced functionality is enabled.

#### Core Configuration

| Parameter                    | Description                                                                     | Default                 | Advanced |
| ---------------------------- | ------------------------------------------------------------------------------- | ----------------------- | -------- |
| `CLAMAV_SETUP_TYPE`          | Auto Configure Configuration each startup - Set to `MANUAL` to disable          | `AUTO`                  |          |
| `CLAMD_SETUP_TYPE`           | Auto Configure ClamD configuration each startup                                 | `$CLAMAV_SETUP_TYPE`    |          |
| `FRESHCLAM_SETUP_TYPE`       | Auto Configure ClamD configuration each startup                                 | `$CLAMAV_SETUP_TYPE`    |          |
| `DATA_PATH`                  | Base Folder for Data Files                                                      | `/data/`                |          |
| `CONFIG_PATH`                | Folder for Config Files                                                         | `/config/`              |          |
| `DEFINITIONS_PATH`           | Folder for Virus Definitions                                                    | `${DATA_PATH}`          |          |
| `CLAMD_CONFIG_FILE`          | Clamd Configuration file                                                        | `clamd.conf`            |          |
| `CLAMD_LOCAL_SOCKET`         | Clamd Socket Name                                                               | `/run/clamd/clamd.sock` |          |
| `CLAMD_TEMP_PATH`            | Camd Temp Location                                                              | `/tmp/clamd/`           |          |
| `CONCURRENT_DATABASE_RELOAD` | Enable non-blocking (multi-threaded/concurrent) database reloads.               | `TRUE`                  |          |
| `ENABLE_CLAMD`               | Enable ClamD Daemon                                                             | `TRUE`                  |          |
| `FRESHCLAM_DATABASES`        | Comma seperated list of additional definitions eg                               |                         |          |
|                              | `http://www.rfxn.com/downloads/rfxn.ndb,http://www.rfxn.com/downloads/rfxn.hdb` |                         |          |
| `LISTEN_PORT`                | ClamD TCP Socket Listen port                                                    | `3310`                  |          |
| `LOG_TYPE`                   | Log to `file`, `console`, `both`, or `none`                                     | `FILE`                  |          |
| `LOG_FILE_CLAMD`             | ClamD Log File                                                                  | `clamd.log`             |          |
| `LOG_FILE_FRESHCLAM`         | Freshclam Log File                                                              | `freshclam.log`         |          |
| `LOG_PATH`                   | Logfile locations                                                               | `/logs/`                |          |
| `LOG_VERBOSE`                | Enable Verbosity in Logs                                                        | `FALSE`                 |          |


#### Virus Definitions Configuration

| Parameter                      | Description                                                                          | Default | Advanced |
| ------------------------------ | ------------------------------------------------------------------------------------ | ------- | -------- |
| `ENABLE_DEFINITIONS_UPDATE`    | Enable Automatic Definitions Updating                                                | `TRUE`  |          |
| `DEFINITIONS_UPDATE_FREQUENCY` | How often to check for new Definitions in minutes                                    | `1440`  | x        |
| `DEFINITIONS_UPDATE_BEGIN`     | What time to do the first dump. Defaults to immediate. Must be in one of two formats |         |          |
|                                | Absolute HHMM, e.g. `2330` or `0415`                                                 |         |          |
|                                | Relative +MM, i.e. how many minutes after starting the container,                    |         |          |
|                                | e.g. `+0` (immediate), `+10` (in 10 minutes), or `+90` in an hour and a half         |         |          |

#### Virus Scanning Settings

| Parameter                      | Description                                      | Default          | Advanced |
| ------------------------------ | ------------------------------------------------ | ---------------- | -------- |
| `DISABLE_CERT_CHECK`           | Disable PE Cert Checks                           | `TRUE`           |          |
| `ENABLE_ALGORITHMIC_DETECTION` | Enable Algorithmic Detection                     | `TRUE`           |          |
| `ENABLE_BYTECODE`              | Enable Bytecode Checks                           | `TRUE`           |          |
| `ENABLE_DETECT_PUA`            | Detect PUA                                       | `TRUE`           |          |
| `ENABLE_PHISHING_SCAN_URLS`    | Scan URLs for Phishing                           | `TRUE`           |          |
| `ENABLE_PHISHING_SIGNATURES`   | Scan for signatures related to Phishing          | `TRUE`           |          |
| `ENABLE_SCAN_ARCHIVE`          | Scan Archives                                    | `TRUE`           |          |
| `ENABLE_SCAN_ELF`              | Scan ELF files                                   | `TRUE`           |          |
| `ENABLE_SCAN_HTML`             | Scan HTML Files                                  | `TRUE`           |          |
| `ENABLE_SCAN_MAIL`             | Scan Mail Files                                  | `TRUE`           |          |
| `ENABLE_SCAN_OLE2`             | Scan OLE2 Files                                  | `TRUE`           |          |
| `ENABLE_SCAN_PDF`              | Scan PDF Files                                   | `TRUE`           |          |
| `ENABLE_SCAN_PE`               | Scan PE Files                                    | `TRUE`           |          |
| `ENABLE_SCAN_SWF`              | Scan SWF Files                                   | `TRUE`           |          |
| `EXCLUDE_PUA`                  | Comma Seperated Values of PUA formats to exclude | `NetTool,PWTool` |          |
| `INCLUDE_PUA`                  | Comma Seperated Values of PUA formats to exclude | `(null)`         |          |

#### Scanning Limits

| Parameter             | Description               | Default | Advanced |
| --------------------- | ------------------------- | ------- | -------- |
| `MAX_EMBEDDEDPE`      | Max filesize Embedded PE  | `10M`   | x        |
| `MAX_FILE_SIZE`       | Max file to scan          | `25M`   | x        |
| `MAX_FILES`           | Max files to scan         | `10000` | x        |
| `MAX_HTMLNORMALIZE`   | Max HTML Normalize        | `10M`   | x        |
| `MAX_HTMLNOTAGS`      | Max HTML No Tags          | `2M`    | x        |
| `MAX_ICONSPE`         | Max IconsPE               | `100`   | x        |
| `MAX_PARTITIONS`      | Max Partitons to Scan     | `50`    | x        |
| `MAX_RECHWP3`         | Max Recursive HWP3        | `16`    | x        |
| `MAX_RECURSION`       | Max Folder Recursion      | `16`    | x        |
| `MAX_SCAN_SIZE`       | Max Scan Size             | `100M`  | x        |
| `MAX_SCRIPTNORMALIZE` | Max Script Normalize Scan | `5M`    | x        |
| `MAX_THREADS`         | Max Scanning Threads      | `10`    | x        |
| `MAX_ZIPTYPERCG`      | Max Zip type Recursive    | `1M`    | x        |
| `PCRE_MATCH_LIMIT`    | PCRE Match Limit          | `10000` | x        |
| `PCRE_MAX_FILE_SIZE`  | PCRE Max File Size        | `25M`   | x        |
| `PCRE_RECMATCH_LIMIT` | PCRE REcursive Max Limit  | `2000`  | x        |
| `STREAM_MAX_LENGTH`   | Max stream size to scan   | `25M`   | x        |

#### Alerting Settings

| Parameter                        | Description                                                        | Default | Advanced |
| -------------------------------- | ------------------------------------------------------------------ | ------- | -------- |
| `ENABLE_ALERT_ENCRYPTED_ARCHIVE` | Alert on encrypted archives (.zip, .7zip, .rar)                    | `FALSE` |          |
| `ENABLE_ALERT_ENCRYPTED_DOC`     | Alert on encrypted documents (.pdf)                                | `FALSE` |          |
| `ENABLE_ALERT_OLE2_MACROS`       | Alert on OLE2 files containing VBA macros                          | `FALSE` |          |
| `ENABLE_ALERT_EXCEEDS_MAX`       | Alert on files exceeding MAX_FILES, MAX_SCAN_SIZE or MAX_RECURSION | `FALSE` |          |

## Users and Groups

| Type  | Name     | ID    |
| ----- | -------- | ----- |
| User  | `clamav` | 10050 |
| Group | `clamav` | 10050 |

### Networking

| Port   | Protocol | Description          |
| ------ | -------- | -------------------- |
| `3310` | tcp      | ClamD Listening Port |

* * *
## Maintenance

### Shell Access

For debugging and maintenance, `bash` and `sh` are available in the container.

### Manual Definition Updates
Manual Definition Updates can be performed by entering the container and typing `update-now`

## Support & Maintenance

- For community help, tips, and community discussions, visit the [Discussions board](/discussions).
- For personalized support or a support agreement, see [Nfrastack Support](https://nfrastack.com/).
- To report bugs, submit a [Bug Report](issues/new). Usage questions may be closed as not-a-bug.
- Feature requests are welcome, but not guaranteed. For prioritized development, consider a support agreement.
- Updates are best-effort, with priority given to active production use and support agreements.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.
