
# Host Applications Integration Hints

## Host Backend Integration

The first question regarding the integration of a *OpenMicrofrontends* compliant *Microfrontend* should be: Is a *Host Backend* integration required.

In doubt, the answer should be yes, because it has a lot of advantages: 

 * The actual *Microfrontends* are hidden behind the *Host Application*
 * No CORS problems
 * Proper browser cache busting based on the release version or timestamp

A *Host Backend* is definitely necessary if:

 * The *Microfrontend* is not publicly available 
 * The *Microfrontend* requests API proxies
 * You want to use Server-Side Rendering

## Using OpenMicrofrontends Generator 

The following assumes you are using the [OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"}
to generate the Starter and Host Backend Integrations.

### Host Backend Integration

The *Host Backend* integration depends on the programming language and the server framework but usually includes two steps:

 1. Implementing a *setup* object that defines URLs, security and caching
 2. Adding some middleware or filter 

Here as example with an *Express* backend and the *hostBackendIntegrationsNodeJs* template:

```ts
import {MyMicrofrontendBaseSetup} from './_generated/microfrontendHostIntegrations';

export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {
  get microfrontendBaseUrl() {
    return 'http://my-microfrontend.my-test-namespace.svc.cluster.local:8080';
  };

  // Security, Caching, ...
}
```

```ts
import {myMicrofrontendHostIntegrationMiddleware} from './_generated/microfrontendHostIntegrations';

const app = express();

// ...

app.use(myMicrofrontendHostIntegrationMiddleware(
  new MyMicrofrontendBaseSetupImpl()
));
```

### Starting the Microfrontend

For the frontend-side you need to 
 
 * generate a Starter 
 * provide some MessageBus implementation
 * optionally, the [SystemJS loader](https://github.com/systemjs/systemjs){target="_blank"} (see [below](#systemjs))

Example:

```html
<div id="root">
    <!-- Here the Microfrontend will appear -->
</div>
```

```ts
import {startMyMicrofrontend} from './_generated/microfrontendStarters';

const hostElement = document.getElementById('root');

const {close, messages} = await startMyMicrofrontend(hostElement, {
        id: '1',
        config: {
            welcomeMessage: 'Microfrontend Demo!',
        },
        messageBus: globalMessageBus, 
    });

// Send a message to the Microfrontend - type-safe!
messages.publish('ping', { ping: true });
```

### Server-Side Rendering

Currently, SSR is only supported by the *hostBackendIntegrationsNodeJs* template. 

It works like this:

 * In your server-side route you fetch the SSR data (HTML + optional script) and
 * Add it to your HTML template

Here, for example, with *Express*:

```typescript
import {myMicrofrontendServerSideRenderer} from './_generated/microfrontendHostIntegrations';

app.get('index', async (req, res) => {
    try {
        const {contentHtml, headHtml} = await myMicrofrontendServerSideRenderer(req, {
            id: '1',
            config: {
                welcomeMessage: 'Microfrontend Demo!',
            },
        });
        return res.render('index', {
            microfrontend1ContentHtml: contentHtml,
            microfrontend1HeadHtml: headHtml,
        });
    } catch (e) {
        // TODO
    }
});
```

And in the *index* template:

```ejs
<html>
    <head>
        <%-microfrontend1HeadHtml%>
    </head>
    <body>
        <h1>Microfrontend Host Application</h1>
        <!-- Important: There should be no whitespace between the div and pre-rendered content -->
        <div id="root"><%-microfrontend1ContentHtml%></div>
        <!-- Script that starts the Microfrontend -->
        <script src="main.js"></script>
    </body>
</html>
```

!!! danger "Important"
    The *id* passed to the SSR Renderer must be the same as the one used when starting the *Microfrontend* one the frontend side.

### SystemJS

If the *Microfrontend* uses the *moduleSystem* *SystemJS* you have to make sure that [SystemJS loader](https://github.com/systemjs/systemjs) is available in the frontend bevor starting it.

### Host Backend Setup

All examples below are based on the *hostBackendIntegrationsNodeJs* template. But they look very similar for other languages.

#### Internationalization

Optionally, you can set the *language* for the current user.

```ts
export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {

    async getLang(req: IncomingMessage) {
        // Determine the user language
        return 'en';
    }   
}
```
You could also pass it to the Starter in the frontend.

#### User

Optionally, you can set the current user (e.g., the *Microfrontend* needs to show the users name).

```ts
export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {

    async getUser(req: IncomingMessage) {
        // Determine the user 
        return {
            username: 'testUser',
            displayName: 'Test User',
        };
    }   
}
```

#### User Permissions

If the *Microfrontend* requires User Permissions but does not provide a route to determine it, the *Host Application* has to provide them: 

```ts
export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {

    async userPermissionsCalculate(req: IncomingMessage) {
        // Calculate the permissions from scopes or user roles 
        return {
            deleteCustomer: true,
        };
    }   
}
```

#### API Proxies

If the *Microfrontend* requests proxies for external APIs, you have to define the actual target URL:

```ts
export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {

    get apiProxyCustomerApiUrl() {
        return process.env.CUSTOMER_API_URL;
    };
}
```

#### Timeouts

You can change the timeouts for requests to the *Microfrontend Server*:

```ts
export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {

    // Default (e.g., for assets)
    get microfrontendRequestTimeoutSec() {
        return 10; // Default: 5
    }

    get apiProxyTimeoutSec() {
        return 30; // Default: 60
    }

    // This potentially blocks rendering, so it should be low
    get microfrontendSSRTimeoutSec() {
        return 3; // Default: 5
    }
}
```

#### Security

Multiple *Microfrontend* routes can declare security requirements:

 * API proxies
 * The SSR route
 * The *User Permissions* route (it is very likely to do so)

In this case you need to provide the necessary security headers:

```ts
export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {

    async apiProxyRequestCustomerApiSecurityHeaders(req: IncomingMessage) {
        return {
            'x-api-key': '123456',
        };
    }

    async ssrGetSecurityHeaders(req: IncomingMessage) {
        return {
            'x-api-key': '123456',
        };
    }
    
    async userPermissionsRequestGetSecurityHeaders(req: IncomingMessage) {
        return {
            'x-api-key': '123456',
        };
    }
}
```

#### Caching

The generated integration code allows you to cache the following:

 * BuildManifest version or timestamp
 * User Permissions
 * SSR result

!!! tip
    You should at least cache the BuildManifest result for a couple of minutes,
    otherwise there will be a request to the *Microfrontend Server* every time the *Microfrontend* is started.


```ts
export default class MyMicrofrontendBaseSetupImpl implements MyMicrofrontendBaseSetup {
    
    async buildTimestampOrVersionCachePut(tsOrVersion: string) {
        // TODO
    }

    async buildTimestampOrVersionCacheGet() {
        // TODO
        return null;
    }

    // Be careful here, the SSR result may depend on the current user
    async ssrCachePut(key: string, userName: string | undefined, result: object) {
        // TODO
    }

    async ssrCacheGet(key: string, userName: string | undefined) {
        // TODO
        return null;
    }

    // Only use this if calculating the permissions is expensive
    async userPermissionsCachePut(key: string, userPermissions: object) {
        // TODO
    }

    async userPermissionsCacheGet(key: string) {
        // TODO
        return null;
    }
}
```

## Custom Host Application Integration

If you plan to implement a custom *Host Application* integration, or want to add *OpenMicrofrontends* support to your existing Portal,
you should:

 1. Read the [Spec](../spec-schema.md) (most important!)
 2. Check out how the [OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"} works 
 3. Consider the hints below

### Starter

A Starter needs to do the following:

 * Load all assets in the *Description*
 * Determine the Renderer function
 * Determine the required context (as defined [here](https://github.com/Open-Microfrontends/open-microfrontends/blob/master/packages/types/lib/OpenMicrofrontendsRenderer.d.ts#L23){target="_blank"})
 * Call the Renderer with the host element and the context

#### Loading JS

All JS assets must be loaded in the sequence defined in the *Description*:

 * By adding a *script* tag to the HTML head (if no *modulesSystem* defined or *none*)
 * By importing it with *import()* or *System.import()* (depending on the *moduleSystem*)

#### Loading CSS

The CSS assets should just be added via *link* tag to the HTML head (in any order).

#### Determine the Renderer function

Here an example implementation:

```ts
const rendererFunctionName = 'myStarter';
const exportedModules = []; // Empty if no moduleSystem

const rendererFunction =
    exportedModules.find((m) => rendererFunctionName in m)?.[rendererFunctionName] ||
    exportedModules.find((m) => 'default' in m && rendererFunctionName in m.default)?.default?.[rendererFunctionName] ||
    (window as any)[rendererFunctionName];
```

### Modules and importMap

If *Microfrontends* define an *importMap* you have to:

 * Add it to the HTML page
 * Resolve conflicts if *Microfrontends* define the same module name with a different target URL

In case of ESM, because the limitations mentioned [here](microfrontends.md/#esm-limitations), you have to calculate a static *importMap* 
and add it to the page before you start the first *Microfrontend*. 

With [SystemJS](https://github.com/systemjs/systemjs) you can add new *importMaps* dynamically, even after other modules have been loaded.

An algorithm to resolve module name conflicts might work like this:

 * If a module is not in *importMaps.imports* add it there
 * Otherwise, create a scope for every *Microfrontend* asset and add it there
 * For every module added to a scope, create an additional scope and add all modules from the same *importMap*.
   This makes sure that external modules import only modules defined in the same *importMap*

Example:

```yaml title="Microfrontend 1 on mf1.foo.com" 
importMap:
  imports:
    module1: 'https://my-modules.com/module1_1_0_0.js',
    module2: 'https://my-modules.com/module2_1_0_0.js',
```

```yaml title="Microfrontend 2 on mf2.foo.com" 
importMap:
  imports:
    module1: 'https://my-modules.com/module1_1_2_0.js',
    module2: 'https://my-modules.com/module2_1_2_0.js',
```

Would lead to this *importMap*:

```json
{
  "imports": {
    "module1": "https://my-modules.com/module1_1_0_0.js",
    "module2": "https://my-modules.com/module2_1_0_0.js"
  },
  "scopes": {
    "https://mf2.foo.com/public/Microfrontend.js": {
      "module1": "https://my-modules.com/module1_1_2_0.js",
      "module2": "https://my-modules.com/module2_1_2_0.js"
    },
    "https://my-modules.com/module1_1_2_0.js": {
      "module2": "https://my-modules.com/module2_1_2_0.js"
    },
    "https://my-modules.com/module2_1_2_0.js": {
      "module1": "https://my-modules.com/module1_1_2_0.js",
    }
  }
}
```

!!! note
    This algorithm is not perfect, but should work for most cases.

### Browser Caching

To make sure you get the correct asset version you should:

 * Fetch from time to time the *Build Manifest* and append the *version* or *timestamp* property to every asset (?v=1.0.0)
 * Alternatively, if there is no *Build Manifest*, add some query that changes every couple of minutes (at least)

If *moduleSystem* is *ESM* or *SystemJS* you should also add an *importMap* entry for the full asset url (with query) so *imports* from other modules still work:

```json
{
  "imports": {
    "https://mf1.foo.com/public/Microfrontend.js": "https://mf1.foo.com/public/Microfrontend.js?v=1.0.0"
  }
}
```

### Host Backend

Your custom *Host Backend* must provide the following:

 * A route for the *Microfrontend* *setup* with all the context information the backend needs to provide:
    * User
    * User Permissions
    * Language
    * Proxy Paths
    * Host Context
 * A proxy for the assets (optional, but recommended)
 * An API proxy
 * Server-Side Rendering (optional)

#### Asset Proxy

The Asset Proxy should provide a route that forwards GET requests to the asset *basePath* of the *Microfrontends*.

Example:

```<host-application>/microfrontends/microfrontend1/assets/index.js``` → ```https://my-microfrontend.com/public/index.js```

#### API Proxies

An API Proxy should provide a route that forwards everything to the API target URL. 

Example:

```<host-application>/microfrontends/microfrontend1/proxies/bff/customers/1234``` → ```https://my-microfrontend.com/api/customers/1234```

#### Caching

There are several things you could cache in your backend:

 * Build Manifests
 * SSR results 
 * User Permissions 

At the very least, you should cache Build Manifests for a couple of minutes.

#### Host Context

If you want to make the *Microfrontends* aware of some host specifics you can pass a *context.hostContext* object when calling the Renderer.

### Annotations

You might define specific *annotations* for your implementation which *Microfrontends* can, for example, use to control their appearance.
