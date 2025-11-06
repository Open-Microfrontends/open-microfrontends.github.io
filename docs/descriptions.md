
# Descriptions

Here are some detail aspects of *OpenMicrofrontend Descriptions*.

## Assets

The assets section defines:

 * Where to find the JS and CSS assets 
 * How to load them 
 * Hints for browser caching
 * Shared modules

Here is a full example:

```yaml
assets:
  basePath: /public
  buildManifestPath: /build.yaml
  js:
    moduleSystem: ESM
    initial:
      - Microfrontend.js
    importMap:
      imports:
        externalModule1: https://ga.jspm.io/npm:externalModule1@1.2.2/index.js
        externalModule2: https://ga.jspm.io/npm:externalModule2@5.1.8/index.js
  css:
    - styles.css
```

### Asset Names

The asset names in the *Description* **must** remain stable between two deployments, i.e., they must not contain a *hash*.

The reason is that in an environment with rolling updates (like *Kubernetes*), there can be multiple release versions running at the same time.

You should consider to expose a [Build Manifest](#build-manifest-browser-caching) to give the *Host Application* means for cache busting. 

### Asset Paths

The actual path of an asset is calculated by combining the *basePath* with the relative path of the asset.
The *basePath* defaults to */*.

In the example above the initial JS asset would be served at */public/Microfrontend.js*.

In this example:

```yaml
assets:
  js:
    initial:
      - public/assets/Microfrontend.js
```

it would be served at */public/assets/Microfrontend.js*.

### Module System

Possible *moduleSystem* values are:

 * [ESM](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules)
 * [SystemJS](https://github.com/systemjs/systemjs)
 * none

Default is *none* which means the JS assets should just be added as <script\> tags.

!!! note
    There are some limitations when using *ESM*, explained in detail in the [Implementation Hints](implementation-hints/microfrontends.md) section.

### Import Maps (Module Sharing)

If your *Microfrontends* require external (shared) modules, you can define them in the *importMap* section.
Scoped module specifier maps (*scopes*) are not supported.

!!! tip
    *importMaps* are the recommended way to share modules between *Microfrontends* and to reduce bundle sizes. 
    <br/>
    If you use a proprietary solution like [Module Federation](https://module-federation.io){target="_blank"}, there is currently no standardized way
    to declare that in the *Description*. But you could always use [Annotations](#annotations).

!!! note 
    There are some limitations when using *ESM*, explained in detail in the [Implementation Hints](implementation-hints/microfrontends.md) section.

### Build Manifest (Browser Caching)

Since the asset names are stable, the *Host Application* needs some means to make sure the Browser loads new assets after a deployment.

This is where the Build Manifest comes into play. The Build Manifest is a JSON file served by the *Microfrontend Server* which contains either 
a *version* or a *timestamp* property that can be used for cache busting, e.g., by adding it as a query parameter to every asset (?.v=1.0.0). 
 
The simplest approach is to use *package.json* as Build Manifest.

The *buildManifestPath* property is the absolute path to the Build Manifest JSON file.

## rendererFunctionName

The *Renderer* is the function that needs to be called to start the *Microfrontend* in the *Host Frontend*. 

The *rendererFunctionName* property is the name of a function which is either

 * A global variable (window object)
 * A named export of one of the initial modules (if *modulesSystem* is ESM or SystemJS)
 * A property of the default export of one of the initial modules (if *modulesSystem* is ESM or SystemJS)

The *Renderer* needs to satisfy the signature defined [here ](https://github.com/Open-Microfrontends/open-microfrontends/blob/master/packages/types/lib/OpenMicrofrontendsRenderer.d.ts){target="_blank"}.
Typically, a generator (such as [OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"}) is used to create a tailored and type-safe signature.

## Security Schemes

Some routes (paths) on the *Microfrontend Server* might have security requirements.

You can describe *security requirements* with the top-level *securitySchemes* section and then apply it to the route as *security requirements*.

Both concepts have been adopted from [OpenAPI](https://www.openapis.org){target="_blank"} and are described in detail here:
 
 * [SecurityScheme Object](https://spec.openapis.org/oas/v3.1.2.html#security-scheme-object)
 * [SecurityRequirement Object](https://spec.openapis.org/oas/v3.1.2.html#security-requirement-object)

Example:

```yaml
securitySchemes:
  BasicAuth:
    type: http
    scheme: basic
  BearerAuth:
    type: http
    scheme: bearer
  ApiKeyAuth:
    type: apiKey
    in: header
    name: X-API-Key
  OpenID:
    type: openIdConnect
    openIdConnectUrl: https://example.com/.well-known/openid-configuration
microfrontends:
- name: My Microfrontend
  apiProxies:
    bff:
      path: /api
      security:
      - ApiKeyAuth: []
  ssr:
    path: /ssr
    security:
    - ApiKeyAuth: []
```

## User Permissions

If the *Microfrontend* has some functionality that depends on the user and his permissions, this can be described in the *userPermissions* section.

Example:

```yaml
userPermissions:
  permissions:
    - name: showDetails
      description: The authenticated user is permitted to see details
    - name: deletePermitted
      description: The authenticated user is permitted to delete items
  provided:
    path: /permissions
    security:
      - ApiKeyAuth: []
```

The actual permissions need to determined at runtime, either:

 1. By a *Microfrontend Server* route, like in the example above via *provided*
 2. By the *Host Application*

Which option you use depends on the system architecture:

 * If your *Microfrontend* is exposed to third parties and comes with a separate security context the first one is the best choice
 * If your *Microfrontend* is used internally as part of a larger frontend application, it makes sense that the *Host Application* determines the permissions, e.g., role-based
 * If your *Microfrontend* doesn't need to know anything about security and just needs to forward access tokens, the second option also is the better choice. 
   In this case, the *Microfrontend* could be used in completely different environments and security contexts.

If you define a route that determines the user permissions, it must return a JSON object like this for a GET request:

```json
{
  "showDetails": true,
  "deletePermitted": false
}
```

!!! danger "Important"
    User Permission cannot be use to actually *protect* something because they can easily be changed in the browser.
    <br/>
    Their purpose is only to control UI behavior.

## API Proxies

The *Microfrontend* can request API (backend) proxies from the *Host Application*.
A proxy is typically required when:

 * The API is not publicly available
 * The endpoint requires security measures the frontend cannot provide
 * CORS problems want to be avoided

Example:

```yaml
apiProxies:
  proxy1:
    description: Proxy for the internal BFF API
    path: /api
    security:
      - ApiKeyAuth: []
  proxy2:
    description: A proxy for an external API
    targets:
      - url: https://localhost:1234/api
        description: Local
      - url: http://my-api.my-namespace:1234/api
        description: Test environment
```
The target of the proxy can either be the *Microfrontend Server* (BFF) or an external API.

### BFF

To proxy the Backend-for-Frontend (BFF) of your *Microfrontend*, you only have to define the absolute base *path*.

### External API

For External APIs you can provide a list of possible targets. The actual target for the current environment needs to be determined 
by the *Host Application*. And it can be one not listed in the *Description*.

### Usage in the Microfrontend

The *Host Application* will pass an object with the relative proxy paths to the *Renderer*:

```typescript
const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config, apiProxyPaths, permissions} = context;

    // The relative path in *apiProxyPaths.bff* will forward the call to <microfrontend-server>/api
    const response = await fetch(`${apiProxyPaths.bff}/customers/${config.customerId}`);
    
}
```

## Server-Side Rendering

If your *Microfrontend* supports *Hybrid Rendering* (SSR + Hydration) you can tell the *Host Application* where to get the pre-rendered HTML from.

Example: 

```yaml
ssr:
  path: /ssr
  security:
    - ApiKeyAuth: []
```

The *Host Application* can then:

 1. POST the context and config to the route (*ssr.path*) 
 2. Add the returned HTML as innerHTML to the *Microfrontend* container
 3. Start the *Microfrontend* with the flag *serverSideRendered* set to *true*

### Server-Side Renderer 

The *Server-Side Renderer* needs to satisfy the signature defined [here ](https://github.com/Open-Microfrontends/open-microfrontends/blob/master/packages/types/lib/OpenMicrofrontendsServerSideRenderer.d.ts){target="_blank"}.
Typically, a generator (such as [OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"}) is used to create a tailored and type-safe signature.

The *Server-Side Renderer* takes the POST body as input, and its result needs to be returned as the response.

### Usage in the Microfrontend

The *serverSideRendered* flag is passed to the *Renderer*:

```typescript
const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config, serverSideRendered} = context;

    if (serverSideRendered) {
        // Hydrate
    } else {
        // Client-side Rendering
    }
}
```

## Config

The *config* section can be used to define an arbitrary configuration object that needs to be passed to the *Renderer*.
It can be used to change the behavior of the *Microfrontend* or to pass some content IDs.

Example:

```yaml
config:
  schema:
    type: object
    properties:
      customerId:
        type: string
    required:
      - customerId
    additionalProperties: false
  default:
    customerId: '1000'
```

The default config **must** be provided, and it must validate against the *schema*.

!!! tip
    *schema* needs to be a valid JSON schema and the type has to be *object*.

!!! note
    A valid default config is required because this allows *Host Applications* and tools to preview *Microfrontends*.

### Usage in the Microfrontend

```typescript
const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config} = context;
    
    // type: string
    const customerId = config.customerId;
}
```

## Messages

The *messages* section can be used to define a list of messages the *Microfrontend* publishes and/or subscribes to.

This allows an exchange of messages between *Microfrontends* in a type-safe way. 

Example: 

```yaml
messages:
  ping:
    publish: true
    subscribe: true
    schema:     
      type: object
      properties:
        ping:
          const: true
      required:
        - ping
```

This describes an exchange of messages via topic *ping* and a message body that looks like this:

```json
{
  "ping": true
}
```

!!! tip
    *schema* needs to be a valid JSON schema and the type has to be *object*.

### Exchange Messages between Microfrontends

To make sure different *Microfrontends* use compatible messages, it makes sense to provide the content schemas as external JSON files and reference them:

```json
{
  "$id": "https://my-company.com/messages/ping",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "properties": {
    "ping": {
      "const": true
    }
  },
  "additionalProperties": false
}
```

```yaml
messages:
  ping:
    publish: true
    subscribe: true
    schema:
      $ref: './pingMessage.json'
```

### Usage in the Microfrontend

In the *Renderer* you will get a *messageBus* object with type-safe *publish* and *subscribe* methods:

```typescript
const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config, messageBus} = context;

    messageBus.publish('ping', {
        ping: true
    });
    // @ts-expect-error
    message.publish('someOtherTopic', {});
}
```

### Usage in the Host Frontend

The *Host Application* must provide a global Message Bus with generic *publish* and *subscribe* implementations. 

The *Starter* returns a *messages* object with type-safe methods for the opposite direction (so, everything published by the *Microfrontend* can be subscribed to and the other way round).

```typescript
const {close, messages} = await startMyMicrofrontend('http://my-microfrontend.my-namespace:7810', hostElement, {
    id: '1',
    config: {
    },
    messageBus: globalMessageBus,
});

messages.subscribe('ping', (message) => {
    console.log(message);
});

// @ts-expect-error
message.subscribe('someOtherTopic', {});
```

## Annotations

The *annotations* section can be used to add arbitrary meta-data and information for *Host Applications*.

It could, for example, be used to:

 * Provide meta-data for dynamic Cockpits that allows automatic discovery of suitable *Microfrontends* for a specific context 
 * Provide some integration hints, like preferred width 
 * Provide security hints, like how to determine the user permissions in specific environments
 * State some extra requirements

## Validation 

You can use the [JSON schema](spec-schema.md) to validate your *Description*:

```
```yaml
$schema: 'https://open-microfrontends.org/schemas/1-0-0.json'
openMicrofrontends: 1.0.0
```

Or use the [OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"}, which also performs semantic checks:

    omg microfrontends.yaml --validationOnly


<br />

[Example Descriptions](https://github.com/Open-Microfrontends/open-microfrontends/tree/master/packages/schemas/examples){.md-button target="_blank"}

<br />
