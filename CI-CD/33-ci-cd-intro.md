# CI-CD — Continuous Integration, Continuous Delivery 101

As we are progressing with our hourney of learning various DevOps concepts, we have already understood one of the core concepts of DevOps is **Automation**. In this chapter, we will be diving into a new paradigm namely **Contineous Integration and Contineous Delivery/Deployment**.  

## Continuous Intergration

Continunous Integration — CI is a practice adopted in software development phase, where the code developed by a team or an invidual is stored in a central repository. The code in the repository is then run through automated builds and tests phases. The key goals of continuous integration is to find and address bugs quicker, improve software quality, and reduce the time it takes to validate and release new software updates. Every revision that is committed to the central repository triggers an automated build and tests and prepared for a release to production.

CI helps developers' productivity by freeing developers from manual tasks of testing the code the built and by automating the build process post testing. It helps in identifying the bugs early and addressing them adequtely during the softwares' lifecycle, due to frequent autoimated testing cycle. Continuous integration helps your team deliver updates to their customers faster and more frequently.

## Continunous Delivery and/or Continunous Deployment

Another integral part of software delivery lifecycle is adopting a strategy to automatically release the product into the production environment. This is achiived by a concept knownas **Continunous Delivery or Continunous Deployment**, which are related concepts that sometimes get used interchangeably. This is a process, where the code when merged in a central repository is passed through the **CI** phase of testing and building a production grade build. Once, all the tests pass the system pushes the updates directly to the software's users.

Adopting the Continunous Delivery strategy speeds-up the time to market by eliminating the lag between coding and customer value—typically days, weeks, or even months.

## GitOps
GitOps uses Git repositories as a single source of truth to deliver infrastructure as code. Submitted code checks the *CI* process, while the *CD* process checks and applies requirements for things like security, infrastructure as code, or any other boundaries set for the application framework. All changes to code are tracked, making updates easy while also providing version control should a rollback be needed.

The fundamental idea behind GitOps was to make operations automatic for the whole system based on a model of the system which was living outside the system - and Git was where most of us choose to put that model.


## Tools used to CICD 
 
We willbe using the following tools to work with CI/CD:

- [Github Action](https://docs.github.com/en/actions)
- *[Jenkins](https://www.jenkins.io/)*
- [Flux — A tool for GitOps](https://fluxcd.io/flux/)



In this chapter, we will be working on tools like Git, Github for working with a [Version Control and Central repository](../version%20control/) of our code. GitHub Actions for achiving CI/CD in our workflow. We will also be working with some of the Open-Source projects for achiving our GitOps goals, like Flux.  


# Resources:
- [CI/CD: The What, Why and How — Github Article ](https://resources.github.com/ci-cd/)
- [GitHub Actions Tutorial - Basic Concepts and CI/CD Pipeline with Docker — TechWorld with Nana](https://youtu.be/R8_veQiYBjI)
- [What is CI/CD — Red Hat Blog](https://www.redhat.com/en/topics/devops/what-is-ci-cd)