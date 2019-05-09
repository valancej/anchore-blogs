# Enforcing Alpine Linux Docker Images vulnerability (CVE-2019-5021) with Anchore

A security vulnerability affecting the Official Alpine Docker Linux images (>=3.3) contain a NULL password for the `root` user. This particular vulnerability has been tracked as [CVE-2019-5021](https://nvd.nist.gov/vuln/detail/CVE-2019-5021). With over 10 million downloads, Alpine Linux is one of the most popular Linux distributions on Docker Hub. In this post, I will demonstrate an understanding of the issue by taking a closer look at two Alpine Docker images, configure Anchore Engine to identify the risk within the vulnerable image, and give a final output based on an Anchore policy evaluation.

## Finding the issue

In build of the Alpine Docker image (>=3.3) the `/etc/shadow` file show the root user password field entry without a password or lock specifier set. We can see this by running an older Alpine Docker image:

```
# docker run docker.io/alpine:3.4 cat /etc/shadow | head -n1 
root:::0:::::
```

With no `!` or password set, this will now be the condition we wish to check with Anchore. 

To see this condition addressed with the latest version of Alpine, run the following command:

```
# docker run docker.io/alpine:latest cat /etc/shadow | head -n1 
root:!::0:::::
```

## Configuring Anchore secret search analyzer

We will now set up Anchore to search for this particular pattern during image analysis, in order to properly identify the known issue. 

Anchore comes with a number of patterns pre-installed that search for some types of secrets and keys, each with a named pattern that can be matched later in anchore policy definition. We can add a new pattern to the analyzer_config.yaml anchore engine configuration file, and start up anchore with this configuration. The new analyzer_config.yaml for example should have a new patterns added, which we've named 'ALPINE_NULL_ROOT':

```

# Section in analyzer_config.yaml

# Options for any analyzer module(s) that takes customizable input
...
...
secret_search:
  match_params:
    - MAXFILESIZE=10000
    - STOREONMATCH=n
  regexp_match:
...
...
    - "ALPINE_NULL_ROOT=^root:::0:::::$"
```

**Note:** By default, an installation of Anchore comes bundled with a default analzyer_config.yaml file. In order to address this particular issue, modifications will need to be made to the analzyer_config.yaml file as shown above. In order to make sure the configuration changes make their way into your installation of Anchore Engine, create an analyzer_config.yaml file and properly mount it into the Anchore Engine Analyzer Service. 

## Create an Anchore policy specific to this issue

Next, I will create a policy bundle containing a policy rule which explicitly look for any matches found of the above `ALPINE_NULL_ROOT` regex create above. If any matches are found, the Anchore policy evaluation will fail.

```
# Anchore ALPINE_NULL ROOT Policy Bundle

{
    "blacklisted_images": [],
    "comment": "Default bundle",
    "id": "alpinenull",
    "mappings": [
        {
            "id": "c4f9bf74-dc38-4ddf-b5cf-00e9c0074611",
            "image": {
                "type": "tag",
                "value": "*"
            },
            "name": "default",
            "policy_id": "48e6f7d6-1765-11e8-b5f9-8b6f228548b6",
            "registry": "*",
            "repository": "*",
            "whitelist_ids": [
                "37fd763e-1765-11e8-add4-3b16c029ac5c"
            ]
        }
    ],
    "name": "Default bundle",
    "policies": [
        {
            "comment": "System default policy",
            "id": "48e6f7d6-1765-11e8-b5f9-8b6f228548b6",
            "name": "DefaultPolicy",
            "rules": [
                {
                    "action": "STOP",
                    "comment": "Do not allow root user with null password",
                    "gate": "secret_scans",
                    "id": "alpine_null_check",
                    "params": [
                        {
                            "name": "content_regex_name",
                            "value": "ALPINE_NULL_ROOT"
                        }
                    ],
                    "trigger": "content_regex_checks"
                }
            ],
            "version": "1_0"
        }
    ],
    "version": "1_0",
    "whitelisted_images": [],
    "whitelists": [
        {
            "comment": "Default global whitelist",
            "id": "37fd763e-1765-11e8-add4-3b16c029ac5c",
            "items": [],
            "name": "Global Whitelist",
            "version": "1_0"
        }
    ]
}
```

**Note:** The above is an entire policy bundle component which will be needed to effectively evaluate against any analyzed images. The key section within this is the *policies* section, where we are using the *secret_scans* gate with *content_regex_name* and *ALPINE_NULL_ROOT* parameters.

## Conduct policy evaluation

Once this policy has been added and activated to an existing Anchore Engine deployment, we can conduct an analysis and policy evaluation of the vulnerable Alpine Docker image (v3.4) via the following command:

```
# anchore-cli evaluate check docker.io/library/alpine:3.4 --detail

Image Digest: sha256:0325f4ff0aa8c89a27d1dbe10b29a71a8d4c1a42719a4170e0552a312e22fe88
Full Tag: docker.io/library/alpine:3.4
Image ID: b7c5ffe56db790f91296bcebc5158280933712ee2fc8e6dc7d6c96dbb1632431
Status: fail
Last Eval: 2019-05-09T05:02:32Z
Policy ID: alpinenull
Final Action: stop
Final Action Reason: policy_evaluation

Gate                Trigger                     Detail                                                                                                                  Status        
secret_scans        content_regex_checks        Secret search analyzer found regexp match in container: file=/etc/shadow regexp=ALPINE_NULL_ROOT=^root:::0:::::$        stop   
```

In the above output we can explicity see that the secret search analyzer found a regular expression match in the Alpine 3.4 Docker image we've analyzed and we've associated a stop action with this policy rule definition, and the overall result of the policy evaluation has failed. 

Given that Alpine is one of the most widely used Docker images, and the impacted versions of it are particularly recent, it is recommended to update to a new version of the image that is not impacted, or modify the image to disable the root account. 