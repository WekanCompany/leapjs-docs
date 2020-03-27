# Logger

Leap comes with a built-in text-based logger which is used during application bootstrapping and several other circumstances such as displaying caught exceptions \(i.e., system logging\). This functionality is provided via the Logger class in the @leapjs/common package.

You can completely override the default logger or customize the default logger by extending it.

You can also make use of the built-in logger, or create your own custom implementation, to log your own application-level events and messages.

For more advanced logging functionality, you can make use of any Node.js logging package, such as Winston, to implement a completely custom, production grade logging system.

## Custom implementation

You can provide a custom logger implementation that fulfills the ILogger interface to be used by Leap for system logging.

Implementing your own custom logger is straightforward. Simply implement each of the methods of the ILogger interface as shown below. For example,

```typescript
import { ILogger } from '@leapjs/common';

export class MyLogger implements ILogger {
  async log(message: any, context = ''): Promise<void> {
    /_ your implementation _/;
  }

  public async debug(message: any, context = ''): Promise<void> {
    /_ your implementation _/;
  }

  public async warn(message: any, context = ''): Promise<void> {
    /_ your implementation _/;
  }

  public async error(message: any, trace: any, context = ''): Promise<void> {
    /_ your implementation _/;
  }

  public async verbose(message: any, context = ''): Promise<void> {
    /_ your implementation _/;
  }
}
```

You can then supply an instance of MyLogger via

```typescript
Logger.use(new MyLogger());
```

This technique, while simple, doesn't utilize dependency injection. This can pose some challenges, particularly for testing, and limit the reusability of MyLogger. For a solution to this, see the Common scenarios section below.

## Extend built-in logger

Rather than writing a logger from scratch, you may be able to meet your needs by extending the built-in Logger class and overriding selected behavior of the default implementation.

```typescript
import { Logger } from '@leapjs/common';

export class MyLogger extends Logger {
  error(message: string, trace: string) {
    // add your tailored logic here
    super.error(message, trace);
  }
}
```

## Common scenarios

For most real world applications, you'll want to take advantage of dependency injection. For example, you may want to inject other services such as configuration into your logger to customize it and in turn inject your custom logger into other controllers and/or services.

To enable dependency injection for your custom logger, use the LeapApplication container to resolve dependencies and create a class instance. For example, to pass in your logger instance to leap, you can

```typescript
const application: LeapApplication = new LeapApplication(...);
const container = application.getContainer();

this.logger.instance = container.resolve<MyLogger>(MyLogger)

Logger.use(this.logger.instance)
```

You could also directly inject your logger into other classes using `@inject`. For more information take a look at the Dependency Injection section

## Using the logger for application logging

We can combine several of the techniques above to provide consistent behavior and formatting across both Leap system logging and our own application event/message logging. In this section, we'll show an example of achieve this

```typescript
import { Injectable, ILogger } from '@leapjs/common';

@Injectable()
export class MyLogger extends ILogger {...}
```

Next, @inject your custom logger, like this:

```typescript
import { Injectable } from '@leapjs/common';
import { MyLogger } from './my-logger';

@Injectable()
export class AuthService {
  constructor(@inject(Logger) private readonly logger: Logger) {
  ...
  }

  login(): User {
    ...
    this.logger.warn(`Invalid credentials provided ${request.body}`, 'Authentication');
    ...
  }
}
```

The second parmeter in the `this.logger.warn` simply provides the context in which the logger is being used.

