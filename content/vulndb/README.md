# Anchore Enterprise 2.1 Feature Series: Enhanced Vulnerability Data

With the release of [Anchore Enterprise 2.1](https://anchore.com/announcing-anchore-enterprise-2-1/) (based on [Anchore Engine 0.5.0](https://anchore.com/opensource/)), Enterprise customers will now receive access to enhanced vulnerability data from Risk Based Security's VulnDB for increased fidelity, accuracy, and live-ness of image vulnerability scanning results. 

Open source software components and dependencies are increasingly making up a vast majority of software applications. Along with the increased usage of OSS comes the inherent security risks these packages present. 

Recognizing that container images need an added layer of security, Anchore conducts a deep image inspection and analysis to uncover what software components are inside of the image, and generate a detailed manifest that includes packages, configuration files, language modules, and artifacts. Following analysis, user-defined acceptance policies are evaluated against the analyzed data to cerfity the container images.

## Viewing Enhanced Vulnerability Results

Just as in previous releases, Anchore Enterprise users can view vulnerability results for image in the UI. 

**Below is a snapshot of Anchore Enterprise with vulnerable packages identified by VulnDB:**

![alt text](images/ui_vulndb_results.png)