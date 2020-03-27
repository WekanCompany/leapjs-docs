# Queues

Queues are a powerful design pattern that help you deal with common application scaling and performance challenges.

LeapJS currently supports queues using RabbitMQ

## RabbitMQ

RabbitMQ is an open-source and lightweight message broker which supports multiple messaging protocols. It can be deployed in distributed and federated configurations to meet high-scale, high-availability requirements. In addition, it's the most widely deployed message broker, used worldwide at small startups and large enterprises.

Leap provides a simple interface to let you interact with RabbitMQ, below is a simple example

```typescript
import { publisher, receiver } from '@leapjs/queue';

async function defaultMailHandler(args: any): Promise<void> {
  const message: {
    body: IMailOptions;
  } = JSON.parse(args.content);
  ....
  receiver.ack(args);
}

const queueName: string = 'mailer'

await receiver.init(configuration.queue.url);
await publisher.init(configuration.queue.url);

if (receiver.isConnected()) {
  receiver.createQueue(queueName);
}

if (publisher.isConnected()) {
  receiver.listen(queueName, Mail.defaultMailHandler);
}
```

**Note that the defaultMailHandler acknowledges the receipt of the message using `receiver.ack(args)`.**

**Queues are set to not use auto acknowledgement by default and it is upto the consumer/receiver to acknowledge successful receipt of the message**

You can use a publisher to push messages to the queue, while receivers can be used to consume the messages pushed onto the queue by one or more publishers

`publisher` provides `init` to establish a connection and `send` to send data to a specific queue

```typescript
publisher.send(queueName: string, data: any)
```

`receiver` similarly provided `init` to establish a connection and `listen` register specific queue handlers as show in the above example

