# Clair and Anchore

As a customer-facing Solutions Architect at Anchore, I have daily conversations with prospects and existing customers about the challenges they face with their container image workloads. During this discovery stage, I often hear a mix of security and devops tools used to automate, orchestate, secure, and release through the lifecycle of containers. 

### Clair 

One of the tools I hear of quite frequently is CoreOS [Clair](https://github.com/coreos/clair). Clair is an open source project for static analysis of vulnerabilites in container images. Clair collects vulnerability data at intervals and stores them in the database, scrubs container images and indexes the installed software packages. If any vulnerabilities are matched to identified software packages in the images, Clair can send out alerts, reports, or block deployments to production environments. For users looking for this specific functionality, Clair is a perfect solution. Additionaly, for [Quay.io](https://quay.io/) users, Clair security scanning comes baked in. 

At Anchore, we love Clair and it's capabilities, and certainly agree that security, particularly static analysis of container images, is a critcal component to a more mature security posture. Container images are a new artifact, and it is not always known what is inside of them. In addition, developers rather than operations teams are often responsible for creating these container images. Due to the variety in containers the problem of vetting artifacts on the way to production environments is more critical than ever.

### How Anchore can help too

Many of our new users and customers come from Clair, and most often it is due to a key principle we center on at Anchore: *A heavily customizable policy enforcement engine that can evolve over time as needs changes.*

Our most succesful open source users and Enteprise customers have highlighted the above as a hard requirement for their continued success with a container security tool. Often our customers have specific compliance requirements they need to fulfil. These requirements could be detailed in documents like CIS Docker Benchmark or NIST 800-190. Or they could have internal security and policy requirements they need to adhere to. Whatever the case is, the top two responses we get when asking for an ideal solution is typically:

- I'm looking for more than just a list of CVEs
- I'm looking to build customizable policy rules to meet specific compliance needs.

Anchore addresses these issues by providing comprehensive coverage of container image contents that extend beyond vulnerability scanning. This includes secrets scanning, misconfigurations, and compliance best practices. In addition to the identification of more than just operating system packages and third party libraries, Anchore provides the user with action results from a policy engine that can be used to block builds, generate reports, or alert. When it comes to particular industries such as Government, Healthcare, and Financial services, compliance is high on the priority list. Typically, these verticals will have strict policy checks they need enforced and critical solutions that need certification. To accomplish this, leveraging container native security tools that provide a complete suite of checks out of the box, becomes a requirememt. 

### Conclusion

It is clear that the importance of static analysis of container images, in particular identifying known vulnerabilites in software packages, is well known at both Anchore and CoreOS Clair. As a potential user or customer deciding on a container security tool, I recommend uncovering some key bullet points you'd like to see in an ideal solution, and aligning those points with core principles and problems certain vendors solve. 
