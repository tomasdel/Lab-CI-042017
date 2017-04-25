# TnT 06-2017 / CI LABS / Rancher, Traefik, Jenkins

This labs is a very simple example on how to deploy an application on Rancher. We will use [tutum/hello-world](https://hub.docker.com/r/tutum/hello-world/)

During this lab, you will learn how to use Rancher to deploy an image, how to play with Traefik, and how to set  up a very simple Jenkins pipeline in order to have continuous delivery.

First of all, you need an account on Github and an API Token. To generate a token, use "Personal access tokens" on your gihub profile and check repo.

## Rancher vocabulary

https://docs.rancher.com

**stack**: A stack is a group of services. Stacks can be used to group together services that together implement an application.

**services**: Cattle adopts the standard Docker Compose terminology for services and defines a basic service as one or more containers created from the same Docker image.

## URLS

Rancher: https://rancher.tnt-labs.org
Jenkins: http://jenkins-primary.jenkins.traefik.tnt-labs.org

## Deploying application using Rancher UI

1. Create a new stack called ` labs-<user>`

2. Add your first service `app` using ` tutum/hello-world`. Don't forget to **Add TCP health check** on port 80.
![](images/service.png?raw=true)

3. If all is ok, your image is running on your host.
![](images/serviceup.png?raw=true)

4. Now you can check that your container is running using shell or logs view

5. You container is running but not reachable... So we have to configure a loadbalancer. Rancher provides a HAproxy LB but we will use [Traefik](https://traefik.io/) with a wildcard dns entry (on Route53).
Traefik is already deployed, you just have to add labels on your service using upgrade feature:
```
traefik.port: 80
traefik.enable: true
traefik.domain: traefik.tnt-labs.org
```
![](images/labels.png?raw=true)

6. Click on upgrade / finish upgrade in order to clean previous containers

7. Congratulation, your application is available using `<service>.<stack>.traefik.tnt-labs.org`
![](images/app.png?raw=true)

8. You can perform a rolling upgrade using an other image: `dockercloud/hello-world`

## Using Jenkins CI

Now we want to industrialize the deployment part. We will use a Jenkins pipeline in order to upgrade image automatically.

1. Fork the Github project.

2. Go to [Jenkins](http://jenkins-primary.jenkins.traefik.tnt-labs.org) and add your Github token.
![](images/token.png?raw=true)

3. Now create a new Multibranch Pipeline project called `test-XXX-CI`
![](images/multibranchpipeline.png?raw=true)

4. Add Github source
![](images/jenkins_config.png?raw=true)

5. You will see your master branch appear
Jenkins will use the Jenkinsfile present on your git repo to generate 1 job per branch.
This simple pipeline will have 2 steps: The first part is to configure the rancher-cli, the second part is to deploy your application.

8. Your first pipeline will run automatically. Go on rancher, a new stack is available ` <jenkins_job_name>-<branch>`.
![](images/jenkinsstack.png?raw=true)

9. You can browse your website using [http://helloworld-application.test-snahelou-ci-master.traefik.tnt-labs.org/](http://helloworld-application.test-snahelou-ci-master.traefik.tnt-labs.org/)

10. Add a Github webhook in order to have continuous deployment.
![](images/hook.png?raw=true)

11. Try to change your docker image into your `docker-compose.yml` and check your pipeline on jenkins
![](images/image2.png?raw=true)

12. Congratulation, Rancher performs a rolling update on your service :)
![](images/site2.png?raw=true)

13. Take a look Blue Ocean UI
![](images/blueocean.png?raw=true)

14. Don't forget to delete stack on rancher and your jenkins jobs.

## Requirements

* Two global environment variables (rancher env api):
  * RANCHER_KEY
  * RANCHER_URL

* One global secret RANCHER_SECRET

* Jenkins plugins:
  * Credentials Binding Plugin
  * Folder
  * CloudBees Credentials Plugin
  * Environment Injector Plugin
  * Blue ocean
