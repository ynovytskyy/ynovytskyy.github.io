---
layout: post
title: "Server-side configuration for Angular apps"
---

It's almost always that Angular applications interact with server-side
and many times they need some sort of configuration coming from the
server that we don't want to hardcode in the client. More specific, but very
common case, especially in more complex situations, is that Angular app is
configured with server-side endpoints using such configuration mechanism.
Service registry coming as configuration from the server allows decoupling
between the Angular client and server-side.

# Requirements
Things that we need to figure out:
- Server-side endpoint available at a predefined URL. We choose
`/uiconfig` which the client app will access by convention (hardcoded).
- `ConfigService` to fetch configuration from the `/uiconfig` endpoint and
allow access to configuration values for other services and/or components.
- Limitations of `Router` and `Resolve` in this scenario
- Fetch configuration before execution reaches all interested parties
- Ensure that config is fetched only once even if there is more than
one component on the same page that rely on configuration.

# Server-side config endpoint
Since server-side can be implemented in many languages and technologies, and is
totally independent of Angular we'll just agree that there is an endpoint under
`/uiconfig` URL that is going to return a JSON object with a key-value structure
that will be consumed by the Angular application. It may look as following:

```json
{
    "helloService": "/hello",
    "default": "Hello World"
}
```

> **It's important** to note that `/uiconfig` becomes the **only** URL that is
generated on the server-side and **all** the other parts parts of the
application including `index.html` can be served as static resources.

# Angular ConfigService
Our Angular application will have a `ConfigService` service to provide
configuration values to all interested parties like components and other
services.

```javascript
const CONFIG_URL = '/uiconfig';
export class Config {}

@Injectable()
export class ConfigService {
  private config: Config;

  constructor(private http: HttpClient) {}

  fetchConfig(): Observable<Config> {
    return this.http.get<Config>(CONFIG_URL).map((res) => {
      this.config = res;
      return res;
    });
  }

  getConfig(): Config { return this.config; }
}
```

The `fetchConfig()` function is the one that does the HTTP request to get
configuration from the `/uiconfig` endpoint. And, as it's using an Observable,
it will eventually when resolved assign the returned HTTP result to the internal
`config` field that can be accessed via the `getConfig()` function of the
service.

For `ConfigService` to be available via injection to other service and
components it's wired up in the `providers` section of the `app.module.ts`:

```javascript
import { ConfigService } from './config/config';
@NgModule({
  ....
  providers: [
    ConfigService,
    ....
  ],
  ....
})
export class AppModule { }
```

# Initializing Angular app with configuration from serve-side
Now the interesting part. We need the `ConfigService` to fetch configuration
before any of the dependent components start their execution. In more
technical terms the Observable returned by `fetchConfig()` needs to resolve
before any call to `getConfig()` is done.

## Limitations of Router and Resolve for config scenario
One of the approaches is to use Router's `Resolve` interface whose contract is
to resolve all `Promise` and `Observable` returned by implementing route guard
before the route starts rendering its matching components.

While this is a feasible approach there are some limitations and drawback to it.
First is that you need `Router`. Not a big one as most apps are big enough and
use routing. Second one is that it does not resolve the guard for root components
(e.g. AppComponent) as the Router's `<router-outlet>` in can't be put in the
`index.html`. The highest in the components hierarchy it can be placed is
`AppComponent` template. The third one is that you'd have to structure your
routes in a specific way with the root route matching `''` and setting resolve
guard. Other routes need to be children for the root one.

## Config resolved before all dependent components reached
Luckily there is one more approach to resolve `Promise` and `Observable` right
at application Initialization. And it does not depend on Router at all. (In fact
Router uses this under the hood.)

`APP_INITIALIZER` is the keyword for our approach and here how it's used:

```javascript
import { ConfigService, initConfig } from './config/config';
@NgModule({
  ....
  providers: [
    ConfigService,
    { provide: APP_INITIALIZER, useFactory: initConfig,
      multi: true, deps: [ConfigService] }
  ],
  ....
})
export class AppModule { }
```

This is registering a provider with `APP_INITIALIZER` token, passing
`initConfig` function as `useFactory` will be used to return an `Observable` or
a `Promise` to be resolved on application initialization. `deps` lists our
`ConfigService` that is going to be passed into the `initConfig` function to
fetch and save config.

`multi` parameter needs to be set to `true` and allows the use of the same
`APP_INITIALIZER` token for multiple providers. We don't use the token to inject
the actual values, but that allows us to have multiple providers being
registered to resolve their respective promises at applications initialization
phase.

```javascript
export function initConfig(configService: ConfigService): () => Promise<any> {
  return (): Promise<any> => {
    return configService.fetchConfig().toPromise();
  };
}
```

## Ensuring config is fetched only once
Using the `APP_INITIALIZER` we hooked into Angular's bootstrapping process that
is going to resolve the `Promise` returned by the `ConfigService.fetchConfig()`
to fetch config from the server-side at `/uiconfig`. When this promise is
resolved config is also save in internal state of the `ConfigService` and is
available by `getConfig()` to all interested parties that can inject the service
in a usual way. So all components are using configuration fetched on
initialization and are not re-fetching it from server side on each `getConfig()`
call.

If the need be, `ConfigService.fetchConfig()` can be called in the runtime
to re-fetch config from the server side, but the synchronization
of the Promise resolution with its usage becomes a responsibility of the calling
party.
