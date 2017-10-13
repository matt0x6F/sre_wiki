Using Jenkins Pipelines with Docker
===

### Summary

The **Pipeline Plugin** for the **Jenkins** Continuous Integration system enables you to define jobs in code, and store them as files within your source code repositories. Version 2 of Jenkins includes Pipeline as standard, and it is easy to add Docker support.

## Overview

Jenkins uses the **Groovy** scripting language for defining jobs with Pipeline, but you can write scripts without learning the details of Groovy. Your job code can call functions from any Jenkins plugin that supports Pipeline.

In most cases, you put the script in a file (called Jenkinsfile by default), and add this to the version control repository for your project. This means that each job only need a minimal configuration on the Jenkins server, which specifies the Jenkinsfile.

You can also write Pipeline scripts directly into the Web interface of Jenkins, but should only use this feature for development and testing.

## Installation

To integrate Docker with Pipeline, install Docker on the Jenkins server and add the CloudBees Docker Pipeline Plugin to the Jenkins installation. Check the compatibility list for the status of other plugins. You do have to ensure that all of the plugins that you reference are installed on every Jenkins instance that you use for that job.

If you maintain Jenkins version 1 systems you will need to install the Pipeline Plugin through the usual process. It is automatically included in all new installations of Jenkins 2.

## Writing A Pipeline Script

Create a file in the version control repository for your project. Set the name of the file as Jenkinsfile unless you need to have multiple build scripts in your repository.

Each file defines a _node_, which represents the instance on which the steps will run. Within the node, write a series of steps. You define a step like this:

```groovy
step("Step name") {
  // commands to run
}
```

You automatically have access to the features of the Groovy scripting language, and the built-in env object gives you access to environment variables. Jenkins plugins provide extra objects that you may use within the script, such as docker. Avoid putting complex commands directly in your Jenkins scripts, though. Write any commands that require multiple options or other logic as separate scripts in an appropriate language, such as UNIX shell or PowerShell, and call them from your Jenkins build script.

## Using Docker Containers

The _docker_ object enables you to run containers from images in Docker repositories, and execute commands within the containers. To run steps inside a container, rather than the host system, declare the steps within an _inside() {}_ block. If you specify an image name for a container without a repository address, the image is downloaded from the _Docker Hub_.

The contents of the workspace are automatically shared with the container. This means that commands will run as you would expect, and post-build steps like _archive_ will work correctly without any changes.

For example, this snippet starts a container from the _ruby_ image tagged _2.3.1_ and runs two steps that execute shell commands within the container:

```groovy
docker.image('ruby:2.3.1').inside {

    stage("Install Bundler") {
      sh "gem install bundler --no-rdoc --no-ri"
    }

    stage("Use Bundler to install dependencies") {
      sh "bundle install"
    }
}
```

You may use multiple containers in a single Jenkins job.

## Using Stored Credentials

Jenkins 2 includes the **Credentials Binding Plugin**, so that you can access stored credentials within your Pipeline scripts.

Create a _withCredentials() {}_ block to retrieve a credential and make it available as an environment variable within the block:

```groovy
withCredentials([
    [$class: 'UsernamePasswordMultiBinding', credentialsId: 'cred1', usernameVariable: 'USERNAME1', passwordVariable: 'PASSWORD1'],
    [$class: 'UsernamePasswordMultiBinding', credentialsId: 'cred2', usernameVariable: 'USERNAME2', passwordVariable: 'PASSWORD2']
  ]) {
  sh 'echo $PASSWORD1'
  echo "${env.USERNAME2}"
}
```

## An Example Pipeline Script

Here is a longer example script:

```groovy
node {
        stage("Main build") {

            checkout scm

            docker.image('ruby:2.3.1').inside {

              stage("Install Bundler") {
                sh "gem install bundler --no-rdoc --no-ri"
              }

              stage("Use Bundler to install dependencies") {
                sh "bundle install"
              }

              stage("Build package") {
                sh "bundle exec rake build:deb"
              }

              stage("Archive package") {
                archive (includes: 'pkg/*.deb')
              }

           }

        }

        // Clean up workspace
        step([$class: 'WsCleanup'])

}
```

## Creating a Job for Running a Pipeline Script

First, define the credentials for the Jenkins CI server to access your source control system in the Web interface using _Manage Jenkins > Credentials_. For example, an SSH key for access to Git repositories.

Create a new job in the Jenkins Web interface, selecting the _Pipeline_ type. In the _Pipeline_ section of the job settings, set the _Definition to Pipeline script from SCM_, the _SCM_ to _Git_, then enter the _Repository URL_ and specify the set of _Credentials_. You do not need to add any other settings.

### Tutorials

The [tutorial for Pipeline](https://jenkins.io/doc/pipeline/) explains how to install and use it. Similarly, the [Docker Pipeline Plugin](https://go.cloudbees.com/docs/cloudbees-documentation/cje-user-guide/chapter-docker-workflow.html) is well-documented.

Once you have written a Pipeline script, check it against the [article on Top 10 Best Practices](https://www.cloudbees.com/blog/top-10-best-practices-jenkins-pipeline-plugin).

### Examples

* [Docker build pipelines, with Jekyll](https://www.cloudbees.com/blog/top-10-best-practices-jenkins-pipeline-plugin)
* [Creating Jenkins pipelines with Ansible](https://wjoel.com/posts/ansible-jenkins-pipeline-part-1.html)
* [Running a JMeter Test via Jenkins Pipeline](https://www.blazemeter.com/blog/running-jmeter-test-jenkins-pipeline)
* [Continuous integration for Android with Jenkins, Docker and AWS](http://flyingtophat.co.uk/blog/2016/07/07/continuous-integration-for-android-with-jenkins-docker-and-aws.html)
* [Jenkins 2.0 and Multi-branch pipeline builds for iOS apps with Fastlane](https://www.quernus.co.uk/2016/04/27/jenkins-2.0-multi-branch-pipeline-ios-fastlane-builds/)

Source: http://www.stuartellis.name/articles/jenkins-pipeline/
