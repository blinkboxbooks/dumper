# Dumper

1. Copy `dumper.properties.sample` to `dumper.properties` and edit to suit your environment.

2. Copy `profiles.yaml.sample` to `profiles.yaml` and edit to suit the queues and exchanges you're going to want to shunt between.

3. Run `rake dump queue_name number_of messages` to execute the dump of the message queues with the number of messages. E.g. `rake dump main_queue 10`.

