# SOC_Lab
This SOC Lab project utilizes a range of free open source resources including Elastic, Kibana, Fleet, MISP, Cortex, The Hive, Shuffle, and more. The goal of his lab is to provide you with a comprehensive hands-on experience in managing a production-like environment of a SIEM (Security Information and Event Management), SOAR (Security Orchestration, Automation, and Response), and EDR (Endpoint Detection and Response) with real world data. Exposing certain endpoint to the public internet to generate real world log data and metrics that can then be used to test alert triggering and case creation/handling. 


## Key Components
In the SOC Lab, we leverage a carefully selected set of free and open-source tools to empower your security operations. Here's an overview of the key components driving this lab:

### <img align="center" src="https://www.elastic.co/apple-icon-57x57.png" height="30px" width="30px">&nbsp;&nbsp;   Elastic, Kibana, Filebeat & Fleet
Elastic, along with the versatile suite it provides, serves as our SIEM solution, responsible for efficiently collecting and storing logs originating from our endpoints. Explore Elastic's range of products here [Link](https://www.elastic.co/).

### <img align="center" src="https://thehive-project.org/assets/ico/favicon.ico" height="30px" width="30px">&nbsp;&nbsp;   The Hive
The Hive acts as an all-in-one SOAR solution, facilitating ticket creation and case investigation based on alerts received from Elastic. Delve deeper into The Hive's capabilities here [Link](https://thehive-project.org/).

### <img align="center" src="https://docs.sekoia.io/assets/playbooks/library/misp.png" height="30px" width="30px"> &nbsp;&nbsp;  MISP
MISP, a high-level threat intelligence platform, plays a pivotal role in automating and analyzing the threats encountered on our endpoints. Learn more about MISP here [Link](https://www.misp-project.org/).

### <img align="center" src="https://thehive-project.github.io/Cortex-Analyzers/images/cortex-logo.png" height="30px" width="30px"> &nbsp;&nbsp;  Cortex
Cortex serves as a central repository of analytical tools, simplifying observable analysis without the need for juggling multiple resources. Explore the capabilities of Cortex here [Link](https://docs.strangebee.com/cortex/).

### <img align="center" src="https://github.com/Shuffle/Shuffle/raw/main/frontend/public/images/Shuffle_logo_new.png" height="30px" width="30px"> &nbsp;&nbsp;  Shuffle
Shuffle, an Automation Suite with an extensive range of connectors to popular software, excels in facilitating seamless notification-to-action across multiple resources. Discover more about Shuffle here [Link](https://shuffler.io/).

### <img align="center" src="https://files.softicons.com/download/social-media-icons/free-social-media-icons-by-uiconstock/png/512x512/AWS-Icon.png" height="30px" width="30px"> &nbsp;&nbsp;  AWS
AWS, an industry leading cloud providing, anchors our lab by utilizing E3 instances to create VMs hosting all our servers. Harness the scalability and flexibility of AWS here [Link](https://aws.amazon.com/).
<br><br>

## Key TakeAways
Throughout this lab, we will will cover a diverse range of skill sets and a multitude of topics relating to Could Security, Cloud hosting/management, Design architecture and more. Our objective for this lab remains the fortification of our environment, adhering meticulously to industry-leading practices to safeguard our lab against potential malicious threats actors. Here's a glimpse of the some of the concepts we explore in this lab:
  - Harnessing the potency of a Key Vault, exemplified by AWS Secret Manager, to securely store and manage sensitive credentials.
  - Fine-tuning and managing the firewall configurations of our VMs, ensuring stringent control over inbound and outbound traffic.
  - Leveraging AWS EC2 Instances for the seamless creation and management of virtual machines, bolstering our infrastructure with cloud-native capabilities.
  - Implementing Identity Access Management (IAM) to control and manage user access to AWS resources.
  - Prioritizing patch management and automation to safeguard against vulnerabilities and ensure the consistent application of security updates.
  - Establishing robust TLS encryption mechanisms using self-signed certificates, fortifying the confidentiality and integrity of data transmission.
  - Designing production-level system architectures that prioritize scalability, reliability, and resilience.
  - Setting up reverse-proxies and configuring cron-jobs to streamline operational tasks and optimize system performance.
  - Implementing best practices for credential handling within our codebase, ensuring the utmost security of sensitive information.
  - Mastering AWS VPC management to create isolated virtual networks with fine-grained control over communication and access.
  - Implementing rigorous security hardening measures by disabling unnecessary services and ports on our host systems, bolstering defence against potential threats.
