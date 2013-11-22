# RabbitShunt

1. Copy `rabbit_shunt.properties.sample` to `rabbit_shunt.properties` and edit to suit your environment.

2. Copy `profiles.yaml.sample` to `profiles.yaml` and edit to suit the queues and exchanges you're going to want to shunt between.

3. Run `rake shunt name_of_profile` to execute the shunt between that queue and that exchange

## BEWARE!

If you shunt from a queue to an exchange which doesn't exist or to an exchange which has no bound queues then you will lose your messages!

This is because Rabbit will make that exchange for you, pump messages to it happily, but they won't end up anywhere. There is no AMQP method to determine if an exchange has bindings, so you must pay attention when writing your exchange details down!
