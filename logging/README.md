# Logging Best Practices

## Table of Contents

[Logging](#logging)
- Intermediate
    + [Exclude sensitive information](#exclude-sensitive-information)
    + [Use structured logging](#use-structured-logging)
- Moderate
    + [Active vs Passive logging](#active-vs-passive-logging)
    + [Logging to stdout and stderr standard output streams](#logging-to-stdout-and-stderr-standard-output-streams)
---
---

## Logging
---

#### Exclude sensitive information

Keep security and compliance requirements in mind:

- Don’t emit sensitive Personal Identifiable Information (PII).
- Don’t emit encryption keys or secrets to your logs.
- Ensure that your company’s privacy policy covers your log data.
- Ensure that your logging add-on provider meets your compliance needs.

#### Use structured logging

Human readable logs are helpful, but structured logs add value, especially as an application’s logs become more complex or as request throughput increases. Structured logging provides a well-defined format for your log entries so you can more easily parse, query, and process them for analytics.JSON is the de facto standard for structured logging, but consider using key=value pairs, XML, or another format for your application logs.

#### Active vs Passive logging

There are two logging strategies: passive and active.

Apps that use passive logging are unaware of the logging infrastructure and log messages to standard outputs. In active logging, the app makes network connections to intermediate aggregators, sends data to third-party logging services, or writes directly to a database or index.

Active logging is considered an antipattern, and it should be avoided.

Read [this](https://12factor.net/logs])for more details.

#### Logging to stdout and stderr standard output streams

The best practice is to write your application logs to the standard output (stdout) and standard error (stderr) streams. You shouldn’t worry about losing these logs, as kubelet, Kubernetes’ node agent, will collect these streams and write them to a local file behind the scenes, so you can access them with Kubernetes.