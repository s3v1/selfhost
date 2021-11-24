# Jenkins

 [Jenkins](https://www.jenkins.io/) is a nice server for devops/CI/CD automation.

## Pre-requisites for Jenkins

You install this first

    sudo apt install openjdk-11-jdk-headless -y

## Install Jenkins

Then follow guide at <https://pkg.jenkins.io/debian-stable/>:

    wget -q -O - 'https://pkg.jenkins.io/debian-stable/jenkins.io.key' | sudo apt-key add - 
    sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
    sudo apt-get update && sudo apt-get install jenkins -y

Jenkins is now installed

## Modify memory usage

Increase or decrease memory by editing this file:

    sudo nano /etc/default/jenkins

The default is:

    #JAVA_ARGS="-Xmx256m"

As an example you can adjusts memory and let it use G1GC it by using:

    JAVA_ARGS="-Xmx512m -XX:+UseG1GC"

Then restart the service:

    sudo service jenkins stop
    sudo service jenkins start

## Use Jenkins

By default Jenkins runs on localhost:8080

You should be able to see the initial admin password, by running:

    cat /var/jenkins_home/secrets/initialAdminPassword