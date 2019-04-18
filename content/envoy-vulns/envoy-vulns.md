# A closer look at Two Envoy Vulnerabilities (CVE-2019-9900 and CVE-2019-9901) and their impact on Istio

In this post, I wanted to take a closer look at recent two vulnerabilites impacting [Envoy Proxy](https://www.envoyproxy.io/) versions 1.9.0 and older ([CVE 2019-9900](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9900) and [CVE 2019-9901](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9901)). Since these two particular CVEs have been identified, they have also been patched in Envoy version 1.9.1. Before diving into the specifics of the vulnerabilites and their impact, I wanted to give some general background on Envoy and Istio.

#### What is Envoy?

Envoy Proxy is a modern, high-performance, small footprint edge and service proxy. Envoy is most comparable to software load balancers such as NGINX and HAProxy. Originally written and deployed at Lyft, Envoy is now an official [graduated project](https://www.cncf.io/announcement/2018/11/28/cncf-announces-envoy-graduation/) of the [Cloud Native Computing Foundation](https://www.cncf.io/). 

For more information on Envoy and a real-world example of its usage in practice, I recommend watching this video: [The Mechanics of Deploying Envoy at Lyft](https://www.youtube.com/watch?v=IeJDjq-COjk).

#### What is Istio?

[Istio](https://istio.io/) is an open source service mesh that layers transparently onto existing distributed applications. It is also a platform, including APIs that let it integrate into any loggin platform, or telemetry or policy system. Istio lets you successfully, and efficiently, run a distributed microservice architecture, and provides a uniform way to secure, coneect, and monitor microservices.

For a clear example on Istio, I recommend watching this video: [What is Istio?](https://www.youtube.com/watch?v=1iyFq2VaL5Y&t=21s)

##### What is a service mesh?

The term service mesh is used to describe the network of microservices that make up such applications and the interactions between them. As a service mesh grows in size and complexity, it can become harder to understand and manage. Its requirements can include discovery, load balancing, failure recovery, metrics, and monitoring. A service mesh also often has more complex operational requirements, like A/B testing, canary rollouts, rate limiting, access control, and end-to-end authentication.

If you are interested in further learning on the concepts of a service mesh and the challenges it is intended to solve, I recommend reading the following post by Hashicorp: [What is a Service Mesh?](https://www.hashicorp.com/resources/what-is-a-service-mesh)

**Istio** provides behavioral insights and operational control over the service mesh as a whole, offering a complete solution to satisfy the diverse requirements of microservice applications.

An **Istio service mesh** is logically split into a data plane and a control plane.

- The data plane is composed of a set of intelligent proxies Envoy deployed as sidecars. These proxies mediate and control all network communication between microservices along with Mixer, a general-purpose policy and telemetry hub.

- The control plane manages and configures the proxies to route traffic. Additionally, the control plane configures Mixers to enforce policies and collect telemetry.

#### Istio and Envoy

**Istio** uses an extended version of the Envoy proxy. Envoy is deployed as a sidecar to the relevant service in the same Kubernetes pod. This deployment allows Istio to extract a wealth of signals about traffic behavior as attributes. Istio can, in turn, use these attributes in Mixer to enforce policy decisions, and send them to monitoring systems to provide information about the behavior of the entire mesh.

### CVE-2019-9900

Envoy expects that its HTTP codecs enforce RFC constraints on valid header values. In particular, it is expected that there are no embedded NUL characters in paths, header values or keys. 

When parsing HTTP/1.x header values, Envoy 1.9.0 and before does not reject embedded zero characters (NUL, ASCII 0x0). This allows remote attackers crafting header values containing embedded NUL characters to potentially bypass header matching rules, gaining access to unauthorized resources.

Based on current information, this only affects HTTP/1.1 traffic. If this is not structurally possible in your network or configuration, then it is unlikely that this vulnerability applies.

View the CVE [Github Issue](https://github.com/envoyproxy/envoy/issues/6434).

### CVE-2019-9901

Envoy does not normalize HTTP URL paths in Envoy 1.9 and before. A remote attacker may craft a path with a relative path, e.g. something/../admin, to bypass access control, e.g. a block on /admin. A backend server could then interpret the unnormalized path and provide an attacker access beyond the scope provided for by the access control policy.

View the CVE [Github Issue](https://github.com/envoyproxy/envoy/issues/6435).

An attacker could bypass access control and could also circumvent DoS prevention system such as rate limiting and authorization for a given backend server.

### Remediation

As mentioned in the introduction, these two vulnerabilites have been patched in Envoy version 1.9.1, and correspondingly in the Envoy builds embedded in Istio 1.1.2 and Istio 1.0.7. The recommend steps for remediation are as follows:

- For Istio 1.1.x deployments: update to a minimum of [Istio 1.1.2](https://istio.io/about/notes/1.1.2/)
- For Istio 1.0.x deployments: update to a minimum of [Istio 1.0.7](https://istio.io/about/notes/1.0.7/)