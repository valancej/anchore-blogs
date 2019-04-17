# A closer look at Two Envoy Vulnerabilities (CVE-2019-9900 and CVE-2019-9901) and their impact on Istio

In this post, I wanted to take a closer look at recent two vulnerabilites impacting [Envoy Proxy](https://www.envoyproxy.io/) versions 1.9 and older ([CVE 2019-9900](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9900) and [CVE 2019-9901](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-9901)). Since these two particular CVEs have been identified, they have also been patched in Envoy version 1.9.1. Before diving into the specifics of the vulnerabilites and their impact, I wanted to give some general background on Envoy and Istio.

#### What is Envoy?

Envoy Proxy is a modern, high-performance, small footprint edge and service proxy. Envoy is most comparable to software load balancers such as NGINX and HAProxy. Originally written and deployed at Lyft, Envoy is now an official [graduated project](https://www.cncf.io/announcement/2018/11/28/cncf-announces-envoy-graduation/) of the [Cloud Native Computing Foundation](https://www.cncf.io/). 