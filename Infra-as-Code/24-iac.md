# What is Infrastructure as Code - IaC
We might have experienced already that automation takes out the confusion and error-prone aspect of manual processes and make them more efficient, and productive. As we strive to follow best practices and try to attempt to automate most of our tasks in the DevOps realm. The philosophy behined the IaC is that the infrastructure has become like data — now the physical layer has been abstracted. It has become just like any other software, unlike being considered as a phycial entity. We can use infrastructure tools the very same way we use software. We can bring in the same best practices from the software domain, such as Continuous Integration/Delivery (CI/CD), Test-Drivern Develiopment, Version Control systems. and apply the same principles to manage our infrastructure. 


Automating Infra provisioning is an important aspect that primarily implies having a codified template for managing servers, operating systems, storage, and other infrastructure components. One of the practices which makes life easy for the Ops team is of automating the process of provisioning and monitoring the infrastructure by codifying it. Instead of repeating the identical steps many times, in IaC some scripts written for specific infrastructure provisioning tasks can be stored in a Version Control system in a modular fashion that can be combined together in different ways through automation. That means, we can model our infrastructure dynamically enabling us to create, modify or even destroy the already created infrastructure with some pre-defined scripts or some CLI commands.


The main premise of the IaC is to make the engineer never log into the server and make some modifications to the server configuration. Instead, make the changes to the definition of the pre-defined infrastructure code allowing the change management pipeline to take over and apply the changes made to the hosted server. As servers are initially created and are configured consistently that align with the dynamic application requirements, we might encounter some configuration drifts. IaC tools help us manage such drifts and keep our *state of the server* as desired and defined in our server configuration code. Kief Morris in his book [Infrastructure as Code](https://www.amazon.in/Infrastructure-as-Code-Kief-Morris/dp/1491924357) defines a hierarchy of techniques, a Sysadmin/DevOps engineer is required to make an efficient effort to automate the process of constantly changing infrastructure: 


- Redesign or reconfigure things so the task isn't necessary at all.
- Implement automation that handles the task transparently, without anyone needing to pay attention.
- Implement a self-service tool so that the task can be done quickly, by someone who doesn't need to know the details, preferably the user or person who needs it done.
- Write documentation so that users can easily carry out the task on their own.


Thus prioritizing the task of a DevOps engineer to build the automation, prevent and handle problems, and make improvements.


In the following sessions, we shall explore the tools used to manage our infrastructure requirements. As it's difficult to manage servers in the cloud well without IaC tools, we shall start with the wonderful tools called **Terraform** and **CloudFormation** which efficiently serve the Infrastructure provisioning and management for the cloud and provision on-prem servers as well. later we shall also look at a configuration management tool **Ansible**, that would help us in *ensuring that systems perform in a manner consistent with expectations over time.

Let's start this exciting new chapter with **Terraform**
# Resources
- [What is Infrastructure as Code - IBM Technolgy on Youtube](https://youtu.be/zWw2wuiKd5o)
- [What is Infrastructure as Code - Techworld with Nana on Youtube](https://youtu.be/POPP2WTJ8es)
- [What is Terraform](https://developer.hashicorp.com/terraform/intro#infrastructure-as-code)
- [Study Guide - Terraform Associate Certification](https://developer.hashicorp.com/terraform/tutorials/certification/associate-study)
- [Terraform Full Course for Beginners - Sandip Das on Youtube](https://www.youtube.com/watch?v=EJ3N-hhiWv0)
- [Ansible 101 by Jeff Geerling on Youtube](https://youtube.com/playlist?list=PL2_OBreMn7FqZkvMYt6ATmgC0KAGGJNAN)
- [AWS CloudFormation](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/Welcome.html)