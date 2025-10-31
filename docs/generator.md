
# OpenMicrofrontends Generator

The [OpenMicrofrontends Generator](https://github.com/Open-Microfrontends/open-microfrontends-generator){target="_blank"} (OMG) is a tool to generate interfaces 
and integration code for an *OpenMicrofrontends Description*.

## Basic Usage

Add to your *package.json*:

```json
{
  "devDependencies": {
    "@open-microfrontends/types": "^1.0.0",
    "@open-microfrontends/open-microfrontends-generator": "^1.0.0"
  }
}
```

Then run the generator like this:

    omg microfrontends.yaml src/_generated -t <template1>,<template2>

## Examples

Generate *Renderer Function* interfaces:

    omg microfrontends.yaml src/_generated -t renderers

Generate *Starter Functions*:

    omg microfrontends.yaml src/_generated -t starters

Generate *Host Backend Integrations* for a *Node.js* based server:

    omg microfrontends.yaml src/_generated -t hostBackendIntegrationsNodeJs

Generate *Host Backend Integrations* for a *Java* based server:

    omg microfrontends.yaml src/main/org/example/_generated -t hostBackendIntegrationsJavaServlet -a packageName=org.example._generated

<br />

[OMG Documentation](https://github.com/Open-Microfrontends/open-microfrontends-generator){.md-button target="_blank"}

<br />
