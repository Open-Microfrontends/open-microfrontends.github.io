---
hide:
  - navigation
  - toc
---

<style>
h1, .md-header {
  display: none;
}
</style>

![OpenMicrofrontends](assets/logo-with-text.png)

<div style="font-size: 1.2em; margin: 30px 0; max-widtfh: 800px;">
The <em>OpenMicrofrontends</em> project aims to provide a formal specification for <em>Microfrontends</em> provided by a server.
Think of it as <a href="https://www.openapis.org" target="_blank">OpenAPI</a> for <em>Microfrontends</em>.
</div>

The specification includes:

* Basic metadata like name, title, description
* The assets (js, css) and how to load them
* The schema of the *config* that can be provided when started
* The schema of messages the *Microfrontend* sends and receives (pub/sub)
* API proxies that are required to access (protected) backends
* Security hints

*OpenMicrofrontends* is not a framework, nor is it owned by any company.
It is a community effort to provide a way to describe *Microfrontends* and how to integrate them in a standardized way.
All code we provide is free and open source, and will be forever.

This specification makes it possible to decouple *Microfrontend* development from their integration and to treat *Microfrontends* exactly like *Microservices*
with only a different formal description.

<br/>

[Getting Started](/getting-started){ .md-button }

<br/>
