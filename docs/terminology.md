
# Terminology

Here is an overview of the terminology and component names used in the specification and in the documentation of this project.

Components for a complete setup with Host Backend integration:

![Components](./assets/components.png)

And here for a simplified, **Browser Standalone**, setup:

![Components Browser Standalone](./assets/components-browser-standalone.png)

!!! note
    With the simplified setup, all components (Microfrontend Server, APIs, ...) must be publicly accessible and CORS must be enabled.
    Furthermore, some API security requirements (e.g., BASIC), user permissions and server-side rendering are not available.

- **Microfrontend:** A Microfrontend is a JavaScript frontend bundled into JS and CSS files.
- **Microfrontend Server:** A *Microfrontend Server* serves the *Microfrontend* assets according to the *OpenMicrofrontend Description*. Additionally, it should expose the *OpenMicrofrontend Description* itself, so Host Backend can dynamically register *Microfrontends* and react to changes.
- **Host Backend:** The backend part of the *Host Application*, which can be used to provide security, proxying of Backend APIs and Server-Side Rendering (SSR).
- **Host Frontend:** The frontend part of the *Host Application*, which is used to start *Microfrontends* and provide means to interact with them.
- **Backend API:** The API a *Microfrontend* uses to fetch data or perform actions. This can be a REST API, GraphQL, or any other type of API that the *Microfrontend* needs to function correctly.

!!! note
    You can find a more detailed description of the components and the terminology in the [Spec](specs/1-0-0.html){target="_blank"}.
