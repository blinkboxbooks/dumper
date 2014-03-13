# Dumper

1. Copy `dumper.properties.sample` to `dumper.properties` and edit to suit your environment.
2. Run `rake dump[queue_name,DLX_name]` to execute the dump of the message queues with the number of messages. E.g. `rake dump[poo]`. Notice there is no space inside the brackets!
3. You will find all the XMLs in a directory that is named after the queue. The xml names are the epoch time they are created at followed by the message number. 

Please note that `queue_name` is required but `DLX_name` is optional and is only needed if there is an `x-dead-letter-exchange` associated with the queue.

