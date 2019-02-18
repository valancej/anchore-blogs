# Understanding the 'dockerfile' policy gate

## Introduction

Understanding how to work with policies is a central component of using Anchore container image inspection and enforcement tools effectively. Anchore policies are how users represent which checks to execute on particular images, and how the results of the policy evaluation should be interpreted. 

At Anchore, Policy Bundles are the unit of policy definition and evaluation. A user may have multiple bundles, but for a policy evaluation, the user must specify a bundle to be evaluated, or default to the bundle currently marked as active. A policy bundle is a single JSON document, composed of **policies**, whitelists, mappings, whitelisted images, and blacklisted images. 

A **policy** is a named set of rules, represented as a JSON object within a Policy Bundle, each of which define a specific check to perform and a resulting action to emit if the check returns a match. These checks are defined as *Gates* that contain *Triggers*. In this post, I will focus on the 'dockerfile' gate and its triggers. 

For reference, there is more detailed documentation on this policy gate located [here](https://anchore.freshdesk.com/support/solutions/articles/36000111104-policy-gate-dockerfile).

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

Example of a policy blacklisting ports 21 and 22: 

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

**no_dockerfile_provided:** This trigger allows checks on the way the image was added, firing if the dockerfile was not explicitly provided at analysis time. This is useful in identifying and qualifying other trigger matches.

### Conclusion and example

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

#### Why is this not so great?

- `FROM node:latest`: This doesn't always have to be bad, but it is something to make developers aware of. If we always use the `latest` tag, we run the risk of our build suddenly breaking if that image tag gets updated. To prevent this from occurring, using a specific tag will help to ensure immutability. Additionally, depending on use of your image, you may not need the full `node:latest` image and its dependencies. Many trusted images have `alpine` version which greatly reduce the total image size, thus reducing the possibility of vulnerabilities in packages, and increasing the time of build. 

    - Example of differences is size (node:6 versus node:alpine)
```
# docker images

node                                                       6                   62905ac2c7de        12 days ago         882MB
node                                                       alpine              ebbf98230a82        2 weeks ago         73.7MB
```

- `EXPOSE 22 3000`: We can see as stated in the comment above the `EXPOSE` instruction, port 22 is only used for testing, therefore we can remove it when building our production ready images. **Note** the placement of the `EXPOSE` instruction (close to the top). `EXPOSE` is a cheap command to run, so it is typically best to declare it as late as possible.

- `# LABEL maintainer="jvalance@email.com"`: We aren't including a `LABEL` instruction. It is generally considered a good practice to add labels for organization, automation, licensing information, etc.

- `# HEALTHCHECK --interval=30s CMD node healthcheck.js`: No `HEALTHCHECK` instruction. Typically, this is useful for telling Docker to periodically check our container health status. Great article on why located [here](https://blog.newrelic.com/engineering/docker-health-check-instruction/).

- `# USER node`: No user defined. I've explained above why this is important to include. Read more about it [here](https://github.com/i0natan/nodebestpractices/blob/master/sections/security/non-root-user.md).

- `ADD example.tar.gz /example`: I've mentioned above why using `COPY` instead of `ADD` is considered better practice.

#### Making the Dockerfile a bit better

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

Many of these mistakes can be checked and validated with Anchore policies and in particular the Dockerfile gate I've discussed in the previous sections. If you are already leveraging Anchore to inspect your container images, I strongly suggest diving into the Dockerfile gate and adjusting it to suit your needs. If not, feel free to take a look at Anchore and how conducting a deep image inspection coupled with flexible polices help users gain insight into the contents of their Docker images and enforce security, compliance, and best-practice requirements. 

