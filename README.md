Rejected
========
Rejected is a AMQP consumer daemon and message processing framework. It allows
for rapid development of message processing consumers by handling all of the
core functionality of communicating with RabbitMQ and management of consumer
processes.

Rejected runs as a master process with multiple consumer configurations that are
each run it an isolated process. It has the ability to collect statistical
data from the consumer processes and report on it.

Features
--------
- Dynamic QoS
- Automatic exception handling including connection management and consumer restarting
- Smart consumer classes that can automatically decode and deserialize message bodies based upon message headers
- Metrics logging and submission to statsd

[![PyPI version](https://badge.fury.io/py/rejected.png)](http://badge.fury.io/py/rejected) [![Downloads](https://pypip.in/d/rejected/badge.png)](https://crate.io/packages/pamqp) [![Build Status](https://travis-ci.org/gmr/rejected.png?branch=master)](https://travis-ci.org/gmr/rejected)

Example Consumer
-----------------
    from rejected import consumer
    import logging

    LOGGER = logging.getLogger(__name__)


    class Test(consumer.Consumer):
        def process(self, message):
            LOGGER.debug('In Test.process: %s' % message.body)

Example Configuration
---------------------
```yaml

    %YAML 1.2
    ---
    Application:
      poll_interval: 10.0
      log_stats: True
      statsd:
        enabled: True
        host: localhost
        port: 8125
      Connections:
        rabbitmq:
          host: localhost
          port: 5672
          user: guest
          pass: guest
          ssl: False
          vhost: /
          heartbeat_interval: 300
      Consumers:
        example:
          consumer: rejected.example.Consumer
          connections: [rabbitmq]
          qty: 2
          queue: generated_messages
          dynamic_qos: True
          ack: True
          max_errors: 100
          config:
            foo: True
            bar: baz

     Daemon:
       user: rejected
       group: daemon
       pidfile: /var/run/rejected/example.%(pid)s.pid

     Logging:
       version: 1
       formatters:
         verbose:
           format: "%(levelname) -10s %(asctime)s %(process)-6d %(processName) -15s %(name) -25s %(funcName) -20s: %(message)s"
           datefmt: "%Y-%m-%d %H:%M:%S"
         syslog:
           format: "%(levelname)s <PID %(process)d:%(processName)s> %(name)s.%(funcName)s(): %(message)s"
       filters: []
       handlers:
         console:
           class: logging.StreamHandler
           formatter: verbose
           debug_only: true
         syslog:
           class: logging.handlers.SysLogHandler
           facility: local6
           address: /var/run/syslog
           #address: /dev/log
           formatter: syslog
       loggers:
         my_consumer:
           level: INFO
           propagate: true
           handlers: [console, syslog]
         rejected:
           level: INFO
           propagate: true
           handlers: [console, syslog]
         urllib3:
           level: ERROR
           propagate: true
       disable_existing_loggers: false
       incremental: false
```
