# Understanding Anchore's 'dockerfile' policy gate

## Introduction

Understanding how to work with policies is a central component of using Anchore container image inspection and enforcement tools effectively. Anchore policies are how users represent which checks to execute on particular images, and how the results of the policy evaluation should be interpreted. 

At Anchore, Policy Bundles are the unit of policy definition and evaluation. A user may have multiple bundles, but for a policy evalution, the user must specify a bundle to be evaluated, or default to the bundle currently marked as active. A policy bundle is a single JSON document, composed of **policies**, whitelists, mappings, whitelisted images, and blacklisted images. 

A **policy** is a named set of rules, represented as a JSON object within a Policy Bundle, each of which define a specific check to perform and a resulting action to emit if the check returns a match. These checks are defined as *Gates* that contain *triggers*. In this post, I will focus on the 'dockerfile' gate and it's triggers. 

## Why do we need a Dockerfile check?

A Dockerfile is a text files that contains all commands, in order, to build a Docker image. In short, it is the blueprint for the container image environment. Since a container is a running instance of an image, in makes sense to incorporate effective mechanisms to check for best practices and potential misconfigurations with the blueprint.

## Dockerfile gate

The Dockerfile gate allows uses to perform checks ont he content of the Dockerfile or Docker history for an image and make policy actions based on the construction of the image, not just it's content. Anchore is either given a Dockerfile or infers one from the Docker image layer history. 