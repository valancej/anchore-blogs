# Clair and Anchore

As a customer-facing Solutions Architect at Anchore, I have daily conversations with prospects and existing customers about the challenges they face with their container image workloads. During this discovery stage, I often hear a mix of security and devops tools used to automate, orchestate, secure, and release through the lifecycle of containers. 

One of the tools I hear of quite frequently is CoreOS [Clair](https://github.com/coreos/clair). Clair is an open source project for static analysis of vulnerabilites in container images. Clair collects vulnerability data at intervals and stores them in the database, scrubs container images and indexes the installed software packages. If any vulnerabilities are matched to identified software packages in the images, Clair can send out alerts, reports, or block deployments to production environments. For users looking for this specific functionality, Clair is a perfect solution. Additionaly, for [Quay.io](https://quay.io/) users, Clair security scanning comes baked in. 

At Anchore, we love Clair and it's capabilities, and certainly agree that security, particularly at the container image stage in the lifecycle, is a critcal component to a more mature security posture. Many of our new users and customers come from Clair, and most often it is due to a key principle we center on at Anchore: *A heavily customizable policy enforcement engine that can evolve over time as needs changes.*

Our most succesful open source users and Enteprise customers have highlighted the above as a hard requirement for their continued success with a container security tool. Often our customers have specific compliance requirements they need to fulfil. These requirements could be detailed in documents like CIS Docker Benchmark or NIST 800-190. Or they could have internal security and policy requirements they need to adhere to. Whatever the case is, the top two responses we get when asking for an ideal solution is typically:

- I'm looking for more than just a list of CVEs (I need to identify secrets, credentials, whitelists, blacklists, etc.)
- I'm looking to build customizable policy rules to meet specific compliance needs.



