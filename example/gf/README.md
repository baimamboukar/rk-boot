# rk-gin
Interceptor & bootstrapper designed for GoFrame framework. Currently, supports bellow functionalities.

| Name | Description |
| ---- | ---- |
| Start with YAML | Start service with YAML config. |
| Start with code | Start service from code. |
| GoFrame Service | GoFrame service. |
| Swagger Service | Swagger UI. |
| Common Service | List of common API available on Gin. |
| TV Service | A Web UI shows application and environment information. |
| Static file handler | A Web UI shows files could be downloaded from server, currently support source of local and pkger. |
| Metrics interceptor | Collect RPC metrics and export as prometheus client. |
| Log interceptor | Log every RPC requests as event with rk-query. |
| Trace interceptor | Collect RPC trace and export it to stdout, file or jaeger. |
| Panic interceptor | Recover from panic for RPC requests and log it. |
| Meta interceptor | Send application metadata as header to client. |
| Auth interceptor | Support [Basic Auth], [Bearer Token] and [API Key] authrization types. |
| RateLimit interceptor | Limiting RPC rate |
| Gzip interceptor | Compress and Decompress message body based on request header. |
| CORS interceptor | Server side CORS interceptor. |
| JWT interceptor | Server side JWT interceptor. |
| Secure interceptor | Server side secure interceptor. |
| CSRF interceptor | Server side csrf interceptor. |

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**  *generated with [DocToc](https://github.com/thlorenz/doctoc)*

- [Installation](#installation)
- [Quick start](#quick-start)
- [YAML Config](#yaml-config)
  - [GoFrame Service](#goframe-service)
  - [Common Service](#common-service)
  - [Swagger Service](#swagger-service)
  - [Prom Client](#prom-client)
  - [TV Service](#tv-service)
  - [Static file handler Service](#static-file-handler-service)
  - [Interceptors](#interceptors)
    - [Log](#log)
    - [Metrics](#metrics)
    - [Auth](#auth)
    - [Meta](#meta)
    - [Tracing](#tracing)
    - [RateLimit](#ratelimit)
    - [CORS](#cors)
    - [JWT](#jwt)
    - [Secure](#secure)
    - [CSRF](#csrf)
  - [Development Status: Stable](#development-status-stable)
  - [Contributing](#contributing)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Installation
`go get github.com/rookie-ninja/rk-boot`

## Quick start

- boot.yaml
```yaml
---
gf:
  - name: greeter       # Required, Name of gin entry
    port: 8080          # Required, Port of gin entry
    enabled: true       # Required, Enable gin entry
    sw:
      enabled: true     # Optional, Enable swagger UI
    commonService:
      enabled: true     # Optional, Enable common service
    tv:
      enabled: true     # Optional, Enable RK TV
```
- main.go
```go
package main

import (
	"context"
	"github.com/rookie-ninja/rk-boot"
)

func main() {
	// Create a new boot instance.
	boot := rkboot.NewBoot()

	// Bootstrap
	boot.Bootstrap(context.Background())

	// Wait for shutdown sig
	boot.WaitForShutdownSig(context.Background())
}
```
```shell script
$ go run main.go
$ curl -X GET localhost:8080/rk/v1/healthy
{"healthy":true}
```
- Swagger: http://localhost:8080/sw
![gin-sw](../../img/gin-sw.png)

- TV: http://localhost:8080/rk/v1/tv
![gin-tv](../../img/gin-tv.png)

## YAML Config
Available configuration
User can start multiple GoFrame servers at the same time. Please make sure use different port and name.

### GoFrame Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.name | The name of gf server | string | N/A |
| gf.port | The port of gf server | integer | nil, server won't start |
| gf.enabled | Enable gf entry or not | bool | false |
| gf.description | Description of gf entry. | string | "" |
| gf.cert.ref | Reference of cert entry declared in [cert entry](https://github.com/rookie-ninja/rk-entry#certentry) | string | "" |
| gf.logger.zapLogger.ref | Reference of zapLoggerEntry declared in [zapLoggerEntry](https://github.com/rookie-ninja/rk-entry#zaploggerentry) | string | "" |
| gf.logger.eventLogger.ref | Reference of eventLoggerEntry declared in [eventLoggerEntry](https://github.com/rookie-ninja/rk-entry#eventloggerentry) | string | "" |

### Common Service
| Path | Description |
| ---- | ---- |
| /rk/v1/apis | List APIs in current GfEntry. |
| /rk/v1/certs | List CertEntry. |
| /rk/v1/configs | List ConfigEntry. |
| /rk/v1/deps | List dependencies related application, entire contents of go.mod file would be returned. |
| /rk/v1/entries | List all Entries. |
| /rk/v1/gc | Trigger GC |
| /rk/v1/healthy | Get application healthy status. |
| /rk/v1/info | Get application and process info. |
| /rk/v1/license | Get license related application, entire contents of LICENSE file would be returned. |
| /rk/v1/logs | List logger related entries. |
| /rk/v1/git | Get git information. |
| /rk/v1/readme | Get contents of README file. |
| /rk/v1/req | List prometheus metrics of requests. |
| /rk/v1/sys | Get OS stat. |
| /rk/v1/tv | Get HTML page of /tv. |

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.commonService.enabled | Enable embedded common service | boolean | false |

### Swagger Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.sw.enabled | Enable swagger service over gf server | boolean | false |
| gf.sw.path | The path access swagger service from web | string | /sw |
| gf.sw.jsonPath | Where the swagger.json files are stored locally | string | "" |
| gf.sw.headers | Headers would be sent to caller as scheme of [key:value] | []string | [] |

### Prom Client
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.prom.enabled | Enable prometheus | boolean | false |
| gf.prom.path | Path of prometheus | string | /metrics |
| gf.prom.pusher.enabled | Enable prometheus pusher | bool | false |
| gf.prom.pusher.jobName | Job name would be attached as label while pushing to remote pushgateway | string | "" |
| gf.prom.pusher.remoteAddress | PushGateWay address, could be form of http://x.x.x.x or x.x.x.x | string | "" |
| gf.prom.pusher.intervalMs | Push interval in milliseconds | string | 1000 |
| gf.prom.pusher.basicAuth | Basic auth used to interact with remote pushgateway, form of [user:pass] | string | "" |
| gf.prom.pusher.cert.ref | Reference of rkentry.CertEntry | string | "" |

### TV Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.tv.enabled | Enable RK TV | boolean | false |

### Static file handler Service
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.static.enabled | Optional, Enable static file handler | boolean | false |
| gf.static.path | Optional, path of static file handler | string | /rk/v1/static |
| gf.static.sourceType | Required, local and pkger supported | string | "" |
| gf.static.sourcePath | Required, full path of source directory | string | "" |

- About [pkger](https://github.com/markbates/pkger)
User can use pkger command line tool to embed static files into .go files.

Please use sourcePath like: github.com/rookie-ninja/rk-gf:/boot/assets

### Interceptors
#### Log
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.loggingZap.enabled | Enable log interceptor | boolean | false |
| gf.interceptors.loggingZap.zapLoggerEncoding | json or console | string | console |
| gf.interceptors.loggingZap.zapLoggerOutputPaths | Output paths | []string | stdout |
| gf.interceptors.loggingZap.eventLoggerEncoding | json or console | string | console |
| gf.interceptors.loggingZap.eventLoggerOutputPaths | Output paths | []string | false |

We will log two types of log for every RPC call.
- zapLogger

Contains user printed logging with requestId or traceId.

- eventLogger

Contains per RPC metadata, response information, environment information and etc.

| Field | Description |
| ---- | ---- |
| endTime | As name described |
| startTime | As name described |
| elapsedNano | Elapsed time for RPC in nanoseconds |
| timezone | As name described |
| ids | Contains three different ids(eventId, requestId and traceId). If meta interceptor was enabled or event.SetRequestId() was called by user, then requestId would be attached. eventId would be the same as requestId if meta interceptor was enabled. If trace interceptor was enabled, then traceId would be attached. |
| app | Contains [appName, appVersion](https://github.com/rookie-ninja/rk-entry#appinfoentry), entryName, entryType. |
| env | Contains arch, az, domain, hostname, localIP, os, realm, region. realm, region, az, domain were retrieved from environment variable named as REALM, REGION, AZ and DOMAIN. "*" means empty environment variable.|
| payloads | Contains RPC related metadata |
| error | Contains errors if occur |
| counters | Set by calling event.SetCounter() by user. |
| pairs | Set by calling event.AddPair() by user. |
| timing | Set by calling event.StartTimer() and event.EndTimer() by user. |
| remoteAddr |  As name described |
| operation | RPC method name |
| resCode | Response code of RPC |
| eventStatus | Ended or InProgress |

- example

```shell script
------------------------------------------------------------------------
endTime=2021-11-01T23:31:01.706614+08:00
startTime=2021-11-01T23:31:01.706335+08:00
elapsedNano=278966
timezone=CST
ids={"eventId":"61cae46e-ea98-47b5-8a39-1090d015e09a","requestId":"61cae46e-ea98-47b5-8a39-1090d015e09a"}
app={"appName":"rk-gf","appVersion":"master-e4538d7","entryName":"greeter","entryType":"GfEntry"}
env={"arch":"amd64","az":"*","domain":"*","hostname":"lark.local","localIP":"192.168.1.104","os":"darwin","realm":"*","region":"*"}
payloads={"apiMethod":"GET","apiPath":"/rk/v1/healthy","apiProtocol":"HTTP/1.1","apiQuery":"","userAgent":"curl/7.64.1"}
error={}
counters={}
pairs={}
timing={}
remoteAddr=localhost:54376
operation=/rk/v1/healthy
resCode=200
eventStatus=Ended
EOE
```

#### Metrics
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.metricsProm.enabled | Enable metrics interceptor | boolean | false |

#### Auth
Enable the server side auth. codes.Unauthenticated would be returned to client if not authorized with user defined credential.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.auth.enabled | Enable auth interceptor | boolean | false |
| gf.interceptors.auth.basic | Basic auth credentials as scheme of <user:pass> | []string | [] |
| gf.interceptors.auth.apiKey | API key auth | []string | [] |
| gf.interceptors.auth.ignorePrefix | The paths of prefix that will be ignored by interceptor | []string | [] |

#### Meta
Send application metadata as header to client.

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.meta.enabled | Enable meta interceptor | boolean | false |
| gf.interceptors.meta.prefix | Header key was formed as X-<Prefix>-XXX | string | RK |

#### Tracing
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.tracingTelemetry.enabled | Enable tracing interceptor | boolean | false |
| gf.interceptors.tracingTelemetry.exporter.file.enabled | Enable file exporter | boolean | RK |
| gf.interceptors.tracingTelemetry.exporter.file.outputPath | Export tracing info to files | string | stdout |
| gf.interceptors.tracingTelemetry.exporter.jaeger.agent.enabled | Export tracing info to jaeger agent | boolean | false |
| gf.interceptors.tracingTelemetry.exporter.jaeger.agent.host | As name described | string | localhost |
| gf.interceptors.tracingTelemetry.exporter.jaeger.agent.port | As name described | int | 6831 |
| gf.interceptors.tracingTelemetry.exporter.jaeger.collector.enabled | Export tracing info to jaeger collector | boolean | false |
| gf.interceptors.tracingTelemetry.exporter.jaeger.collector.endpoint | As name described | string | http://localhost:16368/api/trace |
| gf.interceptors.tracingTelemetry.exporter.jaeger.collector.username | As name described | string | "" |
| gf.interceptors.tracingTelemetry.exporter.jaeger.collector.password | As name described | string | "" |

#### RateLimit
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.rateLimit.enabled | Enable rate limit interceptor | boolean | false |
| gf.interceptors.rateLimit.algorithm | Provide algorithm, tokenBucket and leakyBucket are available options | string | tokenBucket |
| gf.interceptors.rateLimit.reqPerSec | Request per second globally | int | 0 |
| gf.interceptors.rateLimit.paths.path | Full path | string | "" |
| gf.interceptors.rateLimit.paths.reqPerSec | Request per second by full path | int | 0 |

#### CORS
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.cors.enabled | Enable cors interceptor | boolean | false |
| gf.interceptors.cors.allowOrigins | Provide allowed origins with wildcard enabled. | []string | * |
| gf.interceptors.cors.allowMethods | Provide allowed methods returns as response header of OPTIONS request. | []string | All http methods |
| gf.interceptors.cors.allowHeaders | Provide allowed headers returns as response header of OPTIONS request. | []string | Headers from request |
| gf.interceptors.cors.allowCredentials | Returns as response header of OPTIONS request. | bool | false |
| gf.interceptors.cors.exposeHeaders | Provide exposed headers returns as response header of OPTIONS request. | []string | "" |
| gf.interceptors.cors.maxAge | Provide max age returns as response header of OPTIONS request. | int | 0 |

#### JWT
In order to make swagger UI and RK tv work under JWT without JWT token, we need to ignore prefixes of paths as bellow.

```yaml
jwt:
  ...
  ignorePrefix:
   - "/rk/v1/tv"
   - "/sw"
   - "/rk/v1/assets"
```

| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.jwt.enabled | Enable JWT interceptor | boolean | false |
| gf.interceptors.jwt.signingKey | Required, Provide signing key. | string | "" |
| gf.interceptors.jwt.ignorePrefix | Provide ignoring path prefix. | []string | [] |
| gf.interceptors.jwt.signingKeys | Provide signing keys as scheme of <key>:<value>. | []string | [] |
| gf.interceptors.jwt.signingAlgo | Provide signing algorithm. | string | HS256 |
| gf.interceptors.jwt.tokenLookup | Provide token lookup scheme, please see bellow description. | string | "header:Authorization" |
| gf.interceptors.jwt.authScheme | Provide auth scheme. | string | Bearer |

The supported scheme of **tokenLookup** 

```
// Optional. Default value "header:Authorization".
// Possible values:
// - "header:<name>"
// - "query:<name>"
// - "param:<name>"
// - "cookie:<name>"
// - "form:<name>"
// Multiply sources example:
// - "header: Authorization,cookie: myowncookie"
```

#### Secure
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.secure.enabled | Enable secure interceptor | boolean | false |
| gf.interceptors.secure.xssProtection | X-XSS-Protection header value. | string | "1; mode=block" |
| gf.interceptors.secure.contentTypeNosniff | X-Content-Type-Options header value. | string | nosniff |
| gf.interceptors.secure.xFrameOptions | X-Frame-Options header value. | string | SAMEORIGIN |
| gf.interceptors.secure.hstsMaxAge | Strict-Transport-Security header value. | int | 0 |
| gf.interceptors.secure.hstsExcludeSubdomains | Excluding subdomains of HSTS. | bool | false |
| gf.interceptors.secure.hstsPreloadEnabled | Enabling HSTS preload. | bool | false |
| gf.interceptors.secure.contentSecurityPolicy | Content-Security-Policy header value. | string | "" |
| gf.interceptors.secure.cspReportOnly | Content-Security-Policy-Report-Only header value. | bool | false |
| gf.interceptors.secure.referrerPolicy | Referrer-Policy header value. | string | "" |
| gf.interceptors.secure.ignorePrefix | Ignoring path prefix. | []string | [] |

#### CSRF
| name | description | type | default value |
| ------ | ------ | ------ | ------ |
| gf.interceptors.csrf.enabled | Enable csrf interceptor | boolean | false |
| gf.interceptors.csrf.tokenLength | Provide the length of the generated token. | int | 32 |
| gf.interceptors.csrf.tokenLookup | Provide csrf token lookup rules, please see code comments for details. | string | "header:X-CSRF-Token" |
| gf.interceptors.csrf.cookieName | Provide name of the CSRF cookie. This cookie will store CSRF token. | string | _csrf |
| gf.interceptors.csrf.cookieDomain | Domain of the CSRF cookie. | string | "" |
| gf.interceptors.csrf.cookiePath | Path of the CSRF cookie. | string | "" |
| gf.interceptors.csrf.cookieMaxAge | Provide max age (in seconds) of the CSRF cookie. | int | 86400 |
| gf.interceptors.csrf.cookieHttpOnly | Indicates if CSRF cookie is HTTP only. | bool | false |
| gf.interceptors.csrf.cookieSameSite | Indicates SameSite mode of the CSRF cookie. Options: lax, strict, none, default | string | default |
| gf.interceptors.csrf.ignorePrefix | Ignoring path prefix. | []string | [] |

### Development Status: Stable

### Contributing
We encourage and support an active, healthy community of contributors &mdash;
including you! Details are in the [contribution guide](CONTRIBUTING.md) and
the [code of conduct](CODE_OF_CONDUCT.md). The rk maintainers keep an eye on
issues and pull requests, but you can also report any negative conduct to
dongxuny@gmail.com. That email list is a private, safe space; even the zap
maintainers don't have access, so don't hesitate to hold us to a high
standard.

Released under the [MIT License](LICENSE).
