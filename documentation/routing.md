# Routing

### Installation

`npm install @leapjs/router`

### Example controller

```typescript
import {
  Controller,
  Param,
  Body,
  Res,
  Get,
  Post,
  Put,
  Delete,
} from '@leapjs/router';

@Controller('/users')
export class UserController {
  private users: User[];

  @Get()
  getMany(@Res() res: Response) {
    return res.status(HttpStatus.OK).json({ data: { users: this.users } });
  }

  @Get('/:id')
  getOne(@Param('id') id: number) {
    // user = service.getOne
    return res.status(HttpStatus.OK).json({ data: { user } });
  }

  @Post()
  createOne(@Body() user: User) {
    // ...
    // this.users.push(user)
    return res.status(HttpStatus.CREATED).json({ data: { user: user._id } });
  }

  @Put('/:id')
  replaceOne(@Param('id') id: number, @Body() user: any) {
    // ...
    return res.status(HttpStatus.OK).json({ data: { user } });
  }

  @Delete('/:id')
  deleteOne(@Param('id') id: number) {
    // ...
    return res.status(HttpStatus.OK).json({ data: { user: user._id } });
  }
}
```

Then register the controller by passing it to `LeapApplication` during creation

```typescript
application.create(new ExpressAdapter(), {
  prefix: 'v1',
  whitelist: configuration.corsWhitelistedDomains,
  controllers: [UserController, AuthController, RoleController],
  beforeMiddlewares: [helmet(), json(acFilterAttributes)],
  afterMiddlewares: [ErrorHandler],
});
```

### Using Request and Response

You can use the request and response objects by injecting them into the method using `@Req` and `@Res`

```typescript
import { Request, Response } from 'express';
import { Controller, Req, Res, Get } from '@leapjs/router';

@Controller()
export class UserController {
  @Get('/users')
  getMany(@Req() request: Request, @Res() response: Response) {
    return response.send('No users found!');
  }
}
```

**Prefix all controllers routes**

If you want to prefix all your routes, e.g. `/v1` you can use `prefix` option in `LeapApplication`:

```typescript
application.create(new ExpressAdapter(), {
  prefix: 'v1',
  ...
});
```

**Controllers with base routes**

You can prefix all specific controller routes with base route

```typescript
@Controller('/users')
export class UserController {
  // ...
}
```

**Inject route parameters**

You can use `@Param` decorator to inject parameters into your route handler

```typescript
@Get("/users/:id")
getOne(@Param("id") id: string) {
}
```

**Inject query parameters**

To inject query parameters, use the `@QueryParam` decorator

```typescript
@Get("/users")
getUsers(@QueryParam("limit") limit: number) {
}
```

**Inject request body**

To inject request body, use `@Body` decorator:

```typescript
@Post("/users")
saveUser(@Body() user: User) {
}
```

If you provide a class type to the parameter that is decorated with `@Body()`, [class-transformer](https://github.com/typestack/class-transformer) is used internally to automatically create an instance of the given class type from the data received in request body.

**Inject request header parameters**

To inject a specific header, use `@Header` decorator:

```typescript
@Post("/users")
saveUser(@Header("authorization") token: string) {
}
```

If you want to inject all the headers use `@Headers()` decorator.

**Inject cookie parameters**

To get a specific cookie, use `@Cookie` decorator:

```typescript
@Get("/users")
getUsers(@Cookie("token") token: string) {
}
```

If you want to inject all cookies use `@Cookies()` decorator.

**Inject uploaded file**

To inject an uploaded file, use `@File` decorator:

```typescript
@Post("/files")
saveFile(@File("fileName") file: File) {
}
```

You can also specify uploading options to multer this way:

```typescript
const fileUploadOptions = {
  storage: multer.diskStorage({
    destination: (req, file, callback): void => {
      const type = file.fieldname;
      const path = `./storage/${type}`;
      if (!existsSync(path)) {
        mkdirSync(path);
      }
      callback(null, path);
    },
    filename: (req, file, callback): void => {
      callback(null, `${Date.now()}.txt`);
    },
  }),
  fileFilter: (req: any, file: any, callback: any): void => {
    callback(null, file);
  },
  limits: {
    fieldNameSize: 255,
    fileSize: 1024 * 1024 * 2,
  },
};

@Post("/files")
saveFile(@File("fileName", fileUploadOptions) file: File[]) {
}
```

To inject multiple uploaded files use the `@Files` decorator instead.

```typescript
@Files({
  options: fileUploadOptions,
  fields: [
    { name: 'cleaning', maxCount: 3 },
    { name: 'detailing', maxCount: 4 },
  ],
})
```

You can also install multer's file definitions via typings, and use `files: File[]` type instead of `any[]`.

### Using middlewares

You can use any existing express middleware, or create your own. To create your middlewares there is a `@Middleware` decorator, and to use already exist middlewares there are `@UseBefore` and `@UseAfter` decorators.

#### Use existing middleware

There are multiple ways to use middleware. For eg.,

To apply middleware to specific routes

```typescript
import {Controller, Get, UseBefore} from "@leapjs/router";
import Authentication from 'common/middleware/auth';

// ...

@Get("/users/:id")
@UseBefore(accessControl())
@UseBefore(Authentication)
getOne(@Param("id") id: number) {
    // ...
}
```

This way the Authentication and accessControl\(\) middleware will be applied only for `getOne` route handler and will be executed before the route handler runs.

The order In the above example, first the Authentication middleware runs and then passes onto accessControl\(\)

To execute middleware after action use `@UseAfter` decorator instead.

To use middleware per-controller,

```typescript
import { Controller, UseBefore } from '@leapjs/router';
let compression = require('compression');

@Controller()
@UseBefore(accessControl())
@UseBefore(Authentication)
export class UserController {}
```

This way both middleware will be applied to all methods/route handlers of `UserController`, and will be executed before any route handlers. `@UseAfter` can be used similarly.

If you want to use a middleware globally for all controllers you can simply register it during bootstrap:

```typescript
const server = application.create(new ExpressAdapter(), {
  prefix: 'v1',
  controllers: [UserController, AuthController, RoleController],
  beforeMiddlewares: [helmet(), json(acFilterAttributes)],
  afterMiddlewares: [ErrorHandler],
});
```

#### Creating your own middleware

Here is example of creating middleware for express.js:

There are two ways of creating middleware:

First, you can create a simple middleware function:

```typescript
export function RequestLogger(
  request: any,
  response: any,
  next?: (err?: any) => any,
): any {
  ...
  next();
}
```

Second you can create a class, the same class can contain both before and after middleware

```typescript
import { Middleware } from '@leapjs/router';

@Middleware()
class Authentication {
  public before(req: any, res: Response, next: NextFunction): any {
    ...
    next()
  }
  public after(req: any, res: Response, next: NextFunction): any {
    ...
    next()
  }
}
```

Then you can use them using any of the methods mentioned in the previous section.

### Dependency Injection

`@leapjs/router` support DI out of the box and can resolve all controller and middleware dependencies.

That's it, now you can inject your services into your controllers:

```typescript
@Controller()
export class UsersController {
  constructor( @inject(UserService) private readonly userService: UserService;) {}
  ...
}
```

### Reference

**Controller Decorators**

| Signature | Example | Description |
| :--- | :--- | :--- |
| @Controller\(baseRoute: string\) | @Controller\("/users"\) class SomeController | Classes marked with this decorator are registered as controllers and its methods are registered as route handlers. |

**Controller Action Decorators**

| Signature                                   | Example                    | express.js equivalent                                                          |
| :--- | :--- | :--- |
| @Get\(route: string\) | @Get\("/users"\) all\(\) | app.get\("/users", all\) |
| @Post\(route: string\) | @Post\("/users"\) save\(\) | app.post\("/users", save\) |
| @Put\(route: string\) | @Put\("/users/:id"\) update\(\) | app.put\("/users", update\) |
| @Patch\(route: string\) | @Patch\("/users/:id"\) patch\(\) | app.patch\("/users/:id", patch\) |
| @Delete\(route: string\) | @Delete\("/users/:id"\) delete\(\) | app.delete\("/users/:id", delete\) |

**Method Parameter Decorators**

| Signature | Example | Description |
| :--- | :--- | :--- |
| @Req\(\) | getAll\(@Req\(\) request: Request\) | Injects a Request object. |
| @Res\(\) | getAll\(@Res\(\) response: Response\) | Injects a Response object. |
| @Param\(name: string\) | get\(@Param\("id"\) id: number\) | Injects a router parameter. |
| @QueryParam\(name: string\) | get\(@QueryParam\("id"\) id: number\) | Injects a query string parameter. |
| @Header\(name: string\) | get\(@Header\("token"\) token: string\) | Injects a specific request headers. |
| @Headers\(\) | get\(@Headers\(\) params: any\) | Injects all request headers. |
| @Cookie\(name: string\) | get\(@Cookie\("username"\) username: string\) | Injects a cookie parameter. |
| @Cookies\(\) | get\(@Cookies\(\) params: any\) | Injects all cookies. |
| @Body\(\) | post\(@Body\(\) body: any\) | Injects the request body |
| @File\(name: string, options?: UploadOptions\) | post\(@File\("filename"\) file: any\) | Injects an uploaded file. Options accepts a multer options object. |
| @Files\({ options: fileUploadOptions, fields: \[ { name: string, maxCount: number }\]}\) | post\(@Files\({ options: fileUploadOptions, fields: \[ { name: 'album', maxCount: 5 }\]}\) files: any\[\]\) | Injects multiple uploaded files from the response. Options accepts a multer options object. |

**Middleware Decorators**

| Signature | Example | Description |
| :--- | :--- | :--- |
| @Middleware\(\) | @Middleware\(\) class SomeMiddleware {before\(\){} after\(\){}} | Creates a middleware class. |
| @UseBefore\(\) | @UseBefore\(CompressionMiddleware\) | Uses given middleware before a route handler is being executed. |
| @UseAfter\(\) | @UseAfter\(CompressionMiddleware\) | Uses given middleware after route handler is being executed. |

