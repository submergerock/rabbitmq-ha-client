# RabbitMQ HA Client

Some AMQP brokers and specifically RabbitMQ do not support HA out of the box. Rationale for this varies as much as peoples' requirements do, so it's not super surprising that this is the case. However there are basic HA possibilities with RabbitMQ, specifically active-passive brokers using [Pacemaker](http://www.rabbitmq.com/pacemaker.html) or behind a plain old TCP load balancer. For a better description of the latter scenario, please read the [blog post](http://www.joshdevins.net/2010/04/16/rabbitmq-ha-testing-with-haproxy) that started this project. Suffice it to say that in order to make this and many HA topologies work, a client that can do automatic, graceful connection recovery and message redelivery is required. Bonus points of course if you can auto-magically de-duplicate messages in the consumer as is done in [Beetle](http://github.com/xing/beetle).

## Functionality

### Completed

* callbacks to listeners on: connection, connection failure, reconnection, reconnection failure, disconnection (facilitates auto-delete queue recreation)
* creating a new connection while a broker is down (publisher will block until connection is created, so you should probably lazily create your connections)
* publishing messages while a broker is down (publisher will block until connection returns)
* publishing messages after broker has restarted
* consuming messages (non-blocking) using basicGet while a broker is down (consumer will block on basicGet until connection returns)
* consuming messages (non-blocking) using basicGet after a broker has restarted
* consuming messages (blocking) using basicConsume after a broker has restarted (consumer will not notice connection drop at all)
* consistency testing (non-transactional, durable queue):
   * 1000 publishes, 20ms between publishes, ~50 messages/sec, 1 node restart, 0 messages lost
   * 1000 publishes, 10ms between publishes, ~100 messages/sec, 1 node restart, 1 message lost

### To Be Done

* handling of ACKs after a reconnect for messages sent before reconnect
* adding more tests of course
* documentation and examples, specifically what to do on connection and reconnection events (auto-delete queue recreation, etc.)
* handling of transactions after a reconnect for messages sent before reconnect (transaction will fail)
* consistency testing (transactional, durable queue)
* more customizability and tuning for reconnection values
* hook in message receipt path to do message de-duplication
* ability to specify non-blocking option while broker is down/reconnecting (i.e. queue up messages in-memory; this wouldn't make much sense with transactional channels though)

## Usage

Basically this is a drop-in replacement for the standard RabbitMQ ConnectionFactory. Anything that uses that should be able to use the HaConnectionFactory instead. Be certain to review the retry strategies (there are some built-in) if you want custom behaviour on channel failures.

## License

Copyright 2010-2012 [Josh Devins](http://www.joshdevins.net)

Licensed under the Apache License, Version 2.0 (the "License"); you may not use this file except in compliance with the License. You may obtain a copy of the License at

   [http://www.apache.org/licenses/LICENSE-2.0](http://www.apache.org/licenses/LICENSE-2.0)

Unless required by applicable law or agreed to in writing, software distributed under the License is distributed on an "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the License for the specific language governing permissions and limitations under the License. 

## Resources

This RabbitMQ HA client internally makes use of the standard [RabbitMQ AMQP client](http://www.rabbitmq.com/java-client.html) and has borrowed ideas and inspiration from the following sources. Please respect their licenses.

* [RabbitMQ Java messagepaterns library, v0.1.3](http://hg.rabbitmq.com/rabbitmq-java-messagepatterns)
* [Spring Framework v3.0.x](http://static.springsource.org/spring/docs/3.0.x/spring-framework-reference/html/jms.html)
