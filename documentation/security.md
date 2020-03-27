# Security

In this section we cover steps that leap takes to help you to increase the security of your applications.

[Helmet](https://github.com/helmetjs/helmet) is included along with the generated leap projects and starter templates and is automatically configured as a middleware

Leap makes use of the Express [cors](https://github.com/expressjs/cors) package. CORS is enabled by default. You can also pass in a list of whitelisted origins when you create a new `LeapApplication` by passing in an array of allowed origins

```typescript
application.create(new ExpressAdapter(), {
  whitelist: ['http://localhost:3000']
  ...
});
```

