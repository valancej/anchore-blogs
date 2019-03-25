# Identifying secrets with Anchore

Working with containerized applications inherently brings up the question of how to best give these applications access to any sensitive information they may need. This sensitive information can often be in the form of secrets, passwords, or other credentials. This week I decided to explore a couple of bad practices / common shortcuts, and some simple checks to integrate into your testing to promote a more polished security model for container images. 

Historically, I've seen a couple "don'ts" for giving containers access to credentials: 

- Including directly in the source code
- Defining a secret in a Dockerfile with `ENV` instruction

The first should be an obvious no. Including this sensitive information in any source code that is managed by an SCM and eventually baked into the build image, especially any public repositories, is giving anyone access to repos those passwords, keys, creds, etc. I've also seen this placement of secrets inside of the container image using the `ENV` instruction.  `Dockerfiles` are likely managed somewhere, and exposing them in clear text is a practice that should be avoided. A recommended best practice is not only to check for keys and passwords as your images are being built, but implement the proper set of tools for true secrets management. There is an excellent article written by Hashicorp on [Why We Need Dynamic Secrets](https://www.hashicorp.com/blog/why-we-need-dynamic-secrets) which is a good place to start. 

### Using the ENV instruction

Below is a quick example of using the `ENV` instruction to define a variable for called `AWS_SECRET_KEY`. Both AWS Access Keys consist of two parts: an access key ID and a secret access key. These credentials can be used with AWS CLI or API operations are should be kept private. 

```Dockerfile
FROM node:6

RUN mkdir -p /home/node/ && apt-get update && apt-get -y install curl
COPY ./app/ /home/node/app/

ENV AWS_SECRET_KEY="1234q38rujfkasdfgws"
```

For arguments sake, lets pretend I built this image and ran the container with the following command: `docker run --name bad_container -d jvalance/node_critical_fail`

```
$ docker ps | grep bad_container
3bd970d05f16        jvalance/node_critical_fail     "/bin/sh -c 'node /h…"   13 seconds ago      Up 12 seconds         22/tcp, 8081/tcp         bad_container
```

and now `exec` into it with the following: `docker exec -ti --name bad_container /bin/bash` to bring up a shell. Then run the `env` command:

```
# env 
YARN_VERSION=1.12.3
HOSTNAME=3bd970d05f16
PWD=/
HOME=/root
AWS_SECRET_KEY=1234q38rujfkasdfgws
NODE_VERSION=6.16.0
TERM=xterm
SHLVL=1
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
_=/usr/bin/env
```

Now you can see that I've just given anyone with access to this container the ability to grab any environment variable I've defined with the `ENV` instruction.

Similarly with the `docker inspect command`:

```
$ docker inspect 3bd970d05f16 -f "{{json .Config.Env}}"
["PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin","NODE_VERSION=6.16.0","YARN_VERSION=1.12.3","AWS_SECRET_KEY=1234q38rujfkasdfgws"]
```

### Including in source code

Back to our example of a bad Dockerfile:

```Dockerfile
FROM node:6

RUN mkdir -p /home/node/ && apt-get update && apt-get -y install curl
COPY ./app/ /home/node/app/

ENV AWS_SECRET_KEY="1234q38rujfkasdfgws"
```

Here we are copying the contents of the app directory into home/node/app inside the image. Why is this bad? Here's an image of the directory struture:

![alt text](images/directory.png)

and specifically the contents of the credentials file:

```
# credentials

[default]
aws_access_key_id = 12345678901234567890
aws_secret_access_key = 1a2b3c4d5e6f1a2b3c4d5e6f1a2b3c4d5e6f1a2b
[kuber]
aws_access_key_id = 12345678901234567890
aws_secret_access_key = 1a2b3c4d5e6f1a2b3c4d5e6f1a2b3c4d5e6f1a2b
```

Same as I did before, I'll try to find it in the container. 

```
/home/node/app# cat .aws/credentials 
[default]
aws_access_key_id = 12345678901234567890
aws_secret_access_key = 1a2b3c4d5e6f1a2b3c4d5e6f1a2b3c4d5e6f1a2b
api_key = 0349r5ufjdkl45
[kuber]
aws_access_key_id = 12345678901234567890
aws_secret_access_key = 1a2b3c4d5e6f1a2b3c4d5e6f1a2b3c4d5e6f1a2b
```