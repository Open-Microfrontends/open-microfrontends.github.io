
# Microfrontend Implementation Hints

## Exposing microfrontends.yaml

Although it is not mandatory, it is recommended to expose the *Description* file under

 * /microfrontends.yaml
 * or /microfrontends.json

Because we expect *Host Applications* and tooling to appear, that will use this for automatic discovery of available *Microfrontends*.

If you use external JSON schemas in your *Description* file, you should expose them as well.

## Implementing the Renderer

You have to implement a Renderer that satisfies the signature defined [here](https://github.com/Open-Microfrontends/open-microfrontends/blob/master/packages/types/lib/OpenMicrofrontendsRenderer.d.ts){target="_blank"}.

[OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"} can create a typed version of the interface. 

Here for example an implementation for a [Vue.js](https://vuejs.org){target="_blank"}-based *Microfrontend*:

```tsx
import {createApp} from 'vue';
import MyMicrofrontend from './MyMicrofrontend.vue';
import {MyMicrofrontendRenderer} from './_generated/microfrontendRenderers';

const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config, messageBus} = context;

    const mf = createApp(MyMicrofrontend, {
        welcomeMessage: config.welcomeMessage,
    });
    mf.mount(host);

    return {
        onRemove: () => {
            mf.unmount();
        }
    }
}
```

## Exporting the Render Function

There are multiple ways to export the Renderer. 

The generic way it to add it as global variable:

```typescript
import {MyMicrofrontendRendererFunctionName} from './_generated/microfrontendRenderers';

// ...

window[MyMicrofrontendRendererFunctionName] = rendererFn;
```

If *moduleSystem* is *ESM* or *SystemJS* you can also export it:

```ts
import {MyMicrofrontendRendererFunctionName} from './_generated/microfrontendRenderers';

// ...

// Named export (must match renderFunctionName in the Description)
export const startMyMicrofrontend = rendererFn;

// As property of the default export 
// (Recommended, because like this you can use the generated name constant):
export default {
    [MyMicrofrontendRendererFunctionName]: rendererFn,
};
```

## Bundling Assets

You can use any bundler and tooling you want. But you should keep in mind that:

 * The initial asset names must be stable, i.e., they must have a fixed name
 * The output of your bundler must match the *moduleSystem* in the *Description*
 * If your *Microfrontend* supports Server-Side Rendering you should create separate CSS files (so the initial HTML gets rendered properly)

Example configurations for common bundlers:

```js title="webpack"
{
    experiments: {
        // outputModule: true, // ESM
    },
    output: {
        path: resolve(import.meta.dirname, 'dist'),
        filename: 'Microfrontend.js', // Stable
        // module: true, // ESM
        // libraryTarget: 'system', // SystemJS
    },
}
```

```js title="Rollup" 
{
    output: [{
        dir: 'dist',
        entryFileNames: '[name].js', // Stable
        // format: 'es', // ESM
        // format: 'system', // SystemJS
    }],
}
```

```js title="Vite" 
{
    build: {
        rollupOptions: {
            output: [{
                dir: 'dist',
                entryFileNames: '[name].js', // Stable
                format: 'iife',
                // format: 'es', // ESM
                // format: 'system', // SystemJS
            }],
        }
    }
}
```

## Code Splitting

If you want to use dynamic `import()` or other code-splitting measures there are more things to consider:

 * All references to modules/chunks must be relative because the *Host Applications* may change the base path of the assets.
   This means, the *public path* must be determined from the initial assets.
 * All assets not listed in the *Descriptions* should have a hash in their name for cache busting

Add the following to your bundler configuration:

```js title="webpack"
{
    output: {
        // ...
        publicPath: 'auto', // automatic public path, can just be omitted   
        chunkFilename: 'chunk.[contenthash].js',
    },
}
```

```js title="Rollup" 
{
    output: [{
        // ...
        assetFileNames: '[name]-[hash][extname]',
        chunkFileNames: '[name]-[hash].js',
    }],
}
```

```js title="Vite" 
{
    base: '', // automatic public path   
    build: {
        rollupOptions: {
            // ...
            output: [{
                assetFileNames: '[name]-[hash][extname]',
                chunkFileNames: '[name]-[hash].js',
            }],
        }
    }
}
```

Here you can find a [working example](https://github.com/Open-Microfrontends/open-microfrontends-examples/tree/master/browser-standalone-code-splitting){target="_blank"}.

!!! warning
    If you use code splitting with *ESM*, check out the [limitations](#esm-limitations) below.

## Shared Libraries

The recommended way to share libraries and to reduce the bundle size are *importMaps*. 

Keep in mind that:

 * For every *external* you define in your bundler configuration, there needs to be an entry in your *importMap*
 * Sometimes external modules import additional modules, they need to be in the *importMap* as well
 * The *moduleSystem* must be *ESM* or *SystemJS*
 * The modules listed in the *importMap* must match the *moduleSystem* 

Here possible bundler configurations for a [React](https://react.dev){target="_blank"} *Microfrontend* that uses *SystemJS* as *moduleSystem*:

```js title="webpack"
{
    output: {
        path: resolve(import.meta.dirname, 'dist'),
        filename: 'Microfrontend1.js',
        libraryTarget: 'system',
    },
    externalsType: 'system',
    externals: {
        'react': 'react',
        'react-dom': 'react-dom',
        'react-dom/client': 'react-dom/client',
    },
}
```

```js title="Rollup" 
{
    output: {
        dir: 'dist',
            entryFileNames: '[name].js',
            format: 'system',
    },
    external: ['react', 'react-dom', 'react-dom/client'],
}
```

```js title="Vite" 
{
    base: '', // automatic public path   
    build: {
        rollupOptions: {
            // ...
            output: {
                dir: 'dist',
                entryFileNames: '[name].js',
                format: 'system',
            },
            external: ['react', 'react-dom', 'react-dom/client'],
        }
    }
}
```

The matching *importMap* declaration would be:

```yaml
importMap:
    imports:
      react: https://ga.system.jspm.io/npm:react@19.1.1/index.js
      react-dom: https://ga.system.jspm.io/npm:react-dom@19.1.1/index.js
      react-dom/client: https://ga.system.jspm.io/npm:react-dom@19.1.1/client.js
      scheduler: https://ga.system.jspm.io/npm:scheduler@0.26.0/index.js
      process: https://ga.system.jspm.io/npm:process@0.11.10/browser.js
```

And you should use the exact same *importMap* during development in your test page:

```html
<script type="systemjs-importmap">
{
  "imports": {
    "react": "https://ga.system.jspm.io/npm:react@19.1.1/index.js",
    "react-dom": "https://ga.system.jspm.io/npm:react-dom@19.1.1/index.js",
    "react-dom/client": "https://ga.system.jspm.io/npm:react-dom@19.1.1/client.js",
    "scheduler": "https://ga.system.jspm.io/npm:scheduler@0.26.0/index.js",
    "process": "https://ga.system.jspm.io/npm:process@0.11.10/browser.js"
  }
}
</script>
```

Here you can find a [working example](https://github.com/Open-Microfrontends/open-microfrontends-examples/tree/master/browser-standalone-shared-libraries){target="_blank"}.

!!! warning
    Although, *importMaps* are supported for *moduleSystem* *ESM* and *SystemJS*, keep in mind that *ESM* has some serious [limitations](#esm-limitations), see below.

## ESM Limitations

There are serious limitations to ES modules because some browsers still only support a single *importMap* which needs to be preset before the first 
ES module is loaded, see [this issue](https://bugzilla.mozilla.org/show_bug.cgi?id=1916277){target="_blank"}.

This means, *Host Applications* would have to merge all *importMaps* into a single one and add it to the HTML page before the first *Microfrontend* starts. 

This leads to the following limitations with *ESM* when using [OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"}, because it generates Starters which load
*Microfrontend* dynamically:

 * *importMaps* are not supported at all (generation will fail)
 * Code splitting only works if the entry file dynamically imports **all** dependencies, including the framework and components.
   Without this, using query parameters for cache busting (e.g., `Microfrontend.js?v=123`) would break the module resolution.

## Asset Caching

Here some best practices regarding caching:

 * The *Cache-Control* header for your assets should be set to a *max-age* of multiple days, to make sure the browsers and the proxies in between cache it and reduce the load on your server
 * At the same time you should set the *buildManifest* to give the *Host Application* means for cache busting if a new version of the *Microfrontend* gets deployed

## Styling

Styling *Microfrontends* is a notorious challenging topic, because:

 * It should be possible to align the Look&Feel to the *Host Application*
 * You don't want to affect elements outside the *Microfrontend* on the same page 
 * You don't want to load the same CSS rules over and over again (in multiple *Microfrontends*)
 
So, if you can avoid it, you should not bring any CSS rules at all but use CSS classes from a design system provided by the *Host Application*.
Here, an example with [Tailwind CSS](https://tailwindcss.com){target="_blank"}:

```html title="Host Application"
html>
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <style type="text/tailwindcss">
        @theme {
          --color-clifford: #da373d;
        }
    </style>
  </head>
  <body>
    <div id="microfrontend-root"></div>
  </body>
</html>
```

```tsx  title="Microfrontend"
export default function MyMicrofrontend() {
    return (
        <div className="text-gray-700 dark:text-gray-400">
            Hello World!
        </div>    
    )
};
```
If you really need to bring your own CSS rules, make sure to:

 * Provide CSS variables to align the style with the *Host Application*
 * Use a unique prefix for all your selectors to avoid conflicts and unwanted side effects. For example, you could use [this postcss plugin](https://www.npmjs.com/package/postcss-prefix-selector){target="_blank"}.

## Internationalization

The *Host Application* may pass the current language as part of the context:

```ts
const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config, lang} = context;
    
    // Use lang (e.g., "fr") to dynamically select the correct message bundle
}
```

## User Information

The *Host Application* may pass some basic user information:

```ts
const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config, user} = context;

    const displayName =  user?.displayName ?? 'Anonymous';
}

```

## Browser Routing 

The *OpenMicrofrontends* specification does not directly allow to describe routes, because that's something the *Host Application* needs to take care of. 

If you *Microfrontend* uses routes (or in any other way manipulates the location) take into consideration:

 * This significantly reduces the versatility of your *Microfrontend*, because it cannot be combined with other *Microfrontends* that do the same on a *Host Application* page
 * You should provide a way to configure the route prefix to the *Host Application*
 * It might be a good idea to add some annotation with a hint that the *Microfrontend* is manipulating the location

An example for a configurable route prefix and an annotation:

```yaml
# ...
config:
  schema:
    type: object
    properties:
      routePrefix:
        type: string
        description: A route prefix, e.g., /microfrontend1
    required:
      - routePrefix
    additionalProperties: false
  default:
    routePrefix: ''
annotations:
  MANIPULATES_LOCATION: true
```

Here you can find a [working example](https://github.com/Open-Microfrontends/open-microfrontends-examples/tree/master/browser-standalone-routing){target="_blank"}.

## Server-Side Rendering

To enable SSR routing for your *Microfrontend* you have to add two things:
 
 1. A server route that delivers the initial HTML 
 2. The capability to *hydrate* the frontend in your client-side Renderer

The SSR route takes a POST request and passes the body to the server-side Renderer function defined [here](https://github.com/Open-Microfrontends/open-microfrontends/blob/master/packages/types/lib/OpenMicrofrontendsServerSideRenderer.d.ts){target="_blank"}.

[OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"} can create a typed version of the interface.

Here for a [Vue.js](https://vuejs.org){target="_blank"}-based *Microfrontend* on an [Express](https://expressjs.com/)-based server:

```ts
import { createSSRApp } from 'vue';
import { renderToString } from 'vue/server-renderer';
import type {MyMicrofrontendServerSideRenderer} from "../_generated/microfrontendRenderersServerSide";

const rendererFn: MyMicrofrontendServerSideRenderer = async (requestBody) => {
    const {config} = requestBody;

    const microfrontend = createSSRApp(Microfrontend, {
        welcomeMessage: config.welcomeMessage,
    });

    const html = await renderToString(microfrontend);

    return {
        html,
        // injectHeadScript,
    };
};

const app = express();
app.use(express.json());

app.post('/ssr', async (req, res) => {
    const result = await rendererFn(req.body);
    res.json(response);
});
```

!!! tip
    The *injectHeadScript* property can be used to inject some preloaded state for the hydration

In the client-side Renderer you can use the *serverSideRendered* flag to determine if hydration should be performed:

```tsx
import {createApp} from 'vue';
import MyMicrofrontend from './MyMicrofrontend.vue';
import {MyMicrofrontendRenderer} from './_generated/microfrontendRenderers';

const renderer: MyMicrofrontendRenderer = async (host, context) => {
    const {config, serverSideRendered} = context;

    if (serverSideRendered) {
        // Hydrate
        // ...
    } else {
        // Render
        // ...
    }

    // ...
}
```

The corresponding *Description* would look like this:

```yaml
  # ...
  ssr:
    path: /ssr
```

Here you can find a [working example](https://github.com/Open-Microfrontends/open-microfrontends-examples/tree/master/host-backend-integration-ssr){target="_blank"}.
