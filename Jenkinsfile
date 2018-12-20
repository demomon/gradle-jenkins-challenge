pipeline {
    agent {
        kubernetes {
        label 'mypod'
        yaml """apiVersion: v1
kind: Pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 1000
  containers:
  - name: gradle
    image: gradle:4.10-jdk-alpine
    command: ['cat']
    tty: true
"""
        }
    }
    environment {
        rtServer  = ''
        rtGradle  = ''
        buildInfo = ''
        CONTAINER_GRADLE_TOOL = '/usr/bin/gradle'
    }
    stages {
        stage('Test Container') {
            steps {
                container('gradle') {
                    sh 'which gradle'
                    sh 'uname -a'
                    sh 'gradle -version'
                }
            }
        }
        stage('Checkout'){
            steps {
                // git url: 'https://github.com/joostvdg/spring-boot-2-demo.git'
                // git 'https://github.com/jfrog/project-examples.git'
                git 'https://github.com/demomon/gradle-jenkins-challenge.git'
            }
        }
        stage('Preparation') {
            steps {
                script{
                    // create a new Artifactory server using the credentials defined in Jenkins 
                    rtServer = Artifactory.newServer url: 'http://35.204.238.14/artifactory', credentialsId: 'art-admin'

                    // create a new Gradle build
                    rtGradle = Artifactory.newGradleBuild()

                    // set the resolver to the Gradle build to resolve from Artifactory
                    rtGradle.resolver repo:'jcenter', server: rtServer
                    
                    // set the deployer to the Gradle build to deploy to Artifactory
                    rtGradle.deployer repo:'libs-snapshot-local',  server: rtServer

                    // declare that your gradle script does not use Artifactory plugin
                    rtGradle.usesPlugin = false

                    // declare that your gradle script uses Gradle wrapper
                    rtGradle.useWrapper = true
                }
            }
        }
        stage('Build') {
            //run the artifactoryPublish gradle task and collect the build info
            steps {
                script {
                    buildInfo = rtGradle.run buildFile: 'build.gradle', tasks: 'clean build artifactoryPublish'
                    // buildInfo = rtGradle.run rootDir: "gradle-examples/gradle-example", buildFile: 'build.gradle', tasks: 'clean artifactoryPublish'
                }
            }
        }
        stage('Publish Build Info') {
            //collect the environment variables to build info
            //publish the build info
            steps {
                script {
                    buildInfo.env.capture = true
                    rtServer.publishBuildInfo buildInfo
                }
            }
        }
    }
}

