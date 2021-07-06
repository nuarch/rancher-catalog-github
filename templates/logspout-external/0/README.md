# Logspout Rancher External

Customized Logspout stack for Rancher that sends logs to an external (outside of
current Rancher Environment) Logstash Collector.

### Info:

For any services launched from the Rancher UI to use Logspout, please make sure to disable the '-t' [tty] option in the Advanced Options of the service definition.
