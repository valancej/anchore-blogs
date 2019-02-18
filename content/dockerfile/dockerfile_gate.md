# Understanding Anchore's 'dockerfile' policy gate

## Introduction

Understanding how to work with policies is a central component of using Anchore container image inspection and enforcement tools effectively. Anchore policies are how users represent which checks to execute on particular images, and how the results of the policy evaluation should be interpreted. 

At Anchore, Policy Bundles are the unit of policy definition and evaluation. A user may have multiple bundles, but for a policy evalution, the user must specify a bundle to be evaluated, or default to the bundle currently marked as active. A policy bundle is a single JSON document, composed of **policies**, whitelists, mappings, whitelisted images, and blacklisted images. 

A **policy** is a named set of rules, represented as a JSON object within a Policy Bundle, each of which define a specific check to perform and a resulting action to emit if the check returns a match. These checks are defined as *Gates* that contain *Triggers*. In this post, I will focus on the 'dockerfile' gate and it's triggers. 

## Why do we need a Dockerfile check?

A Dockerfile is a text file that contains all commands, in order, to build a Docker image. In short, it is the blueprint for the container image environment. Since a container is a running instance of an image, in makes sense to incorporate effective mechanisms to check for best practices and potential misconfigurations with the blueprint.

## Dockerfile gate

The Dockerfile gate allows uses to perform checks on the content of the Dockerfile or Docker history for an image and make policy actions based on the construction of the image, not just it's content. Anchore is either given a Dockerfile or infers one from the Docker image layer history. 

### The *actual_dockerfile_only* parameter

The actual versus history impacts the semantics of the Dockerfile gate's triggers. To allow explicit control of the differences, most triggers in this gate includes a parameter: *actual_dockerfile_only* that if set to true or false will ensure the trigger check is only done on the source of data specified. If *actual_dockerfile_only* = true, then the trigger will evaluate only if an actual Dockerfile is available for the image and will skip evaluation if not. If *actual_dockerfile_only* is false or omitted, then the trigger will run on the actual Dockerfile if available, or the history data if the Dockerfile was not provided.

### Triggers

**Instruction:** This trigger evaluates instructions found in the Dockerfile.

Example of a policy looking for the presence of the `ADD` instruction: 

```
{
  "action": "WARN",
  "gate": "dockerfile",
  "id": "c35b7509-b0de-4b7a-9749-47380a2f98f2",
  "params": [
    {
      "name": "instruction",
      "value": "ADD"
    },
    {
      "name": "check",
      "value": "exists"
    },
    {
      "name": "actual_dockerfile_only"
      "value": "true"
    }
  ],
  "trigger": "instruction"
}
```

In the above example, if the actual Dockerfile contains the instruction `ADD` a WARN action will result. Generally speaking, using the instruction `COPY` versus `ADD` is considered better practice. Read more about it [here](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#add-or-copy).

**Effective User:** This trigger processes all `USER` directives in the Dockerfile or history to determine which user will be user to run the container by default (assuming no user is set explicitly at runtime). The detected value is then subject to a whitelist or blacklist filter depending on the configured parameters. 

Running containers as root is generally considered to be a bad practice, however adding a `USER` instruction to the Dockerfile to specify a non-root user for the container to run as is a good place to start. If you do need to run as root, you can change the user to root at the beginning of the Dockerfile, then change back to the correct user with a second `USER` instruction. 

Example policy to blacklist root user: 

```
{
  "gate": "dockerfile",
  "trigger": "effective_user", 
  "action": "stop", 
  "parameters": [ 
    {
      "name": "users",
      "value": "root"
    }, 
    {
      "name": "type",
      "value": "blacklist"
    }
  ]
}
```

**Exposed ports:** This trigger processes the set of `EXPOSE` directives in the Dockerfile or history to determine the set of ports that are defined to be exposed (since it can span multiple directives). The detected value is then subject to a whitelist or blacklist filter depending on the configured parameters.

Example of a plicy blacklisting ports 21 and 22: 

```
{
  "gate": "dockerfile",
  "trigger": "exposed_ports", 
  "action": "warn", 
  "parameters": [ 
    {
      "name": "ports",
      "value": "21,22"
    }, 
    {
      "name": "type",
      "value": "blacklist"
    }
  ]
}
```

### An example of a very poorly written Dockerfile

A short example of why writing secure and efficient Dockerfiles is important. You can probably spot a good chunk of issues with the following Dockerfile:

```
FROM node:latest

## port 22 for testing only
EXPOSE 22 3000 

RUN apt-get update
RUN apt-get install -y curl nginx

# LABEL maintainer="jvalance@email.com"

ADD example.tar.gz /example

# HEALTHCHECK --interval=30s CMD node healthcheck.js 
# USER node
```

Many of these mistakes can be checked and validated with Anchore policies and in particular the Dockerfile gate I've discussed above. 

```
FROM node:6.16.0-alpine

LABEL maintainer="jvalance@email.com"

RUN apt-get update && apt-get install -y \
        curl \
        nginx

COPY example.tar.gz /example

HEALTHCHECK --interval=30s CMD node healthcheck.js 
USER node

EXPOSE 3000
```