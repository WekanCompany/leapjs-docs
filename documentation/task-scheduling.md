# Task-Scheduling

Task scheduling allows you to schedule methods/functions to execute at a fixed date/time, at recurring intervals, or once after a specified interval. Commonly, this is often handled by packages like cron at the OS level.

LeapJS provides the `@leapjs/scheduler`, which is a simple way to scheduling tasks.

It is built on top of MongoDB and the `@leapjs/queue` package. The tasks are stored in a separate collection and `@leapjs/queue` is used to ensure tasks are reliably run in a distributed environment.

## Getting started

The scheduler service lets you schedule both one-time and recurring tasks. The scheduler depends on queues to dispatch and run the tasks, so queues must be initiated before the scheduler can be started.

See [queues](https://github.com/WekanCompany/leapjs/wiki/Queues) for more information

You can start the scheduler by

```typescript
await publisher.init(configuration.queue.url);

if (publisher.isConnected()) {
  const scheduler = new Scheduler(publisher);
  scheduler.start();
}
```

Once connected, you can create a queue to be used by the tasks using `receiver.createQueue`.

_We highly recommend you to read_ [_this_](https://www.cloudamqp.com/blog/2017-12-29-part1-rabbitmq-best-practice.html) _article before starting to work with RabbitMQ_

You can find examples for creating different task types below,

```typescript
scheduler
  .createTask(
    // Unique task name
    `Send out registration mail out for ${email}`,
    // Queue to use to dispatch the task
    'mailer',
    // Task type - recurring, once
    'once',
    // Data needed to complete the task, type: any
    data,
    // Start date - Optional
    moment(Date.now())
      .add(5, 'm')
      .toDate(),
  )
  .catch((error: any) => {
    Logger.error(error.message, '', 'Scheduler');
  });
```

The above example, creates a task to be run 5 minutes from the current time. If no start datetime is provided, the task is immediately dispatched to the queue.

```typescript
scheduler
  .createTask(
    // Unique task name
    `Send out report mail out for ${new Date().toISOString()}`,
    // Queue to use to dispatch the task
    'mailer',
    // Task type - recurring, once
    'recurring',
    // Data needed to complete the task, type: any
    data,
    // Start date - Optional
    undefined,
    // End date - Optional
    moment(Date.now())
      .add(12, 'm')
      .toDate(),
    // Repeat interval - Run every night at 12am
    '0 0 0 * * ?',
  )
  .catch((error: any) => {
    Logger.error(error.message, '', 'Scheduler');
  });
```

The repeat interval for recurring tasks, use cron strings to define the intervals.

If a start or end date is provided, the task repeats at the interval defined only within the start and end dates.

```typescript
function mailHandler(args: any): void {
  const body = JSON.parse(args.content);
  ....
  receiver.ack(args);
  Scheduler.endTask(body);
}
```

As mentioned in the queues section, consumers must acknowledge receipt of the message and handle errors. An additional step is required for queued tasks. On successfully completing the task, it upto the task handler to end it using `Scheduler.endTask(body)`.

One-time jobs that are not ended, will be retried automatically.

