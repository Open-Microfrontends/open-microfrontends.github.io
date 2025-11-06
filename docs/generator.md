
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

To generate code run:

    omg <descriptionFile> <outFolder> -t <template1>,<template2>

Arguments:

* *descriptionFile*: The OpenMicrofrontends description (yaml/json)
* *outFolder*: The target folder for generated code (not required for --validationOnly)

Options:

| Option                     | Description                                                         |
|----------------------------|---------------------------------------------------------------------|
| -t, --templates            | A comma separated list of templates to use                          |
| -a, --additionalProperties | A comma separated list of extra properties to pass to the templates |
| -v, --validationOnly       | Only validate given spec file and exit                              |
| --help                     | Usage info                                                          |

## Examples

Validate your *Description*:

    omg microfrontends.yaml --validationOnly 

Generate *Renderers*:

    omg microfrontends.yaml src/_generated -t renderers

Generate *Starters*:

    omg microfrontends.yaml src/_generated -t starters

Generate *Host Backend Integrations* for a *Node.js* based server:

    omg microfrontends.yaml src/_generated -t hostBackendIntegrationsNodeJs

Generate *Host Backend Integrations* for a *Java* based server:

    omg microfrontends.yaml src/main/org/example/_generated -t hostBackendIntegrationsJavaServlet -a packageName=org.example._generated

<br />

[OMG Documentation](https://github.com/Open-Microfrontends/open-microfrontends-generator){.md-button target="_blank"}

<br />
