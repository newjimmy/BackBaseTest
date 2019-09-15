#!/usr/bin/env groovy
import java.util.regex.Matcher
def validateParam(KEY, VAL) {
    if (VAL == null || VAL == '') {
        currentBuild.result = 'ABORTED'
        def err = KEY + " was not defined. Aborted."
        currentBuild.displayName = "#[REPO: ${REPOSITORY}] [BRANCH: ${BRANCH}] - ${err}"
        echo err
        error(err)
    }
    if (KEY == 'BRANCH') {
        if ( !(VAL ==~ /[a-z0-9]([-a-z0-9]*[a-z0-9])?/) ) {
            error("namespace $VAL is not a valid DNS label: a DNS-1123 label must consist of lower case alphanumeric characters or '-', and must start and end with an alphanumeric character (e.g. 'my-name',  or '123-abc', regex used for validation is '[a-z0-9]([-a-z0-9]*[a-z0-9])?)'")
        }
    }
}

pipeline {
    agent {
        node {
            label "docker"
        }
    }

    parameters{
        string(name: 'REPOSITORY', defaultValue: 'test', description: 'Repository name under corporate git like a https://github.com/newjimmy/BackBaseTest')
        string(name: 'BRANCH', defaultValue: 'dev')
    }

    stages {
        stage("Validate params") {
            steps{
                script {
                    validateParam("REPOSITORY", params.REPOSITORY)
                    validateParam("BRANCH", params.BRANCH)
                }
            }
        }

        stage("Checkout repo"){
            steps{
                script {
                    checkout scm: [$class           : 'GitSCM',
                                   userRemoteConfigs: [[url: "https://github.com/newjimmy/${params.REPOSITORY}.git", credentialsId: 'SECURED_CREDENTIALS']],
                                   branches         : [[name: "${params.BRANCH}"]]], changelog: false, poll: false
                }
            }
        }
        // Checking coding style
        stage('SonarQube analysis') {
            def scannerHome = tool 'SonarScanner 4.0';
                    withSonarQubeEnv('My SonarQube Server') {
                                    sh 'mvn clean package sonar:sonar'
                    }
        }

        stage('Build') {
            steps {
                script {
                    withDockerContainer(image: 'maven', args: '-u root:root') {
                        sh '''
                        mvn -B -DskipTests clean package
                        '''
                    }
                }
            }
        }

        stage('Test') {
            steps {
                script {
                    withDockerContainer(image: 'maven', args: '-u root:root') {

                        sh '''
                        mvn test
                        '''
                    }
                }
            }
        }

        stage("Push package") {
             steps {
                 script {
                     withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'ArtifactoryPassword', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD']]) {
                        withDockerContainer(image: 'private-docker-registry.org.com/java-mvn', args: '-u root:root') {

                            sh '''

                                '''
                        }
                     }
                 }
             }
        }
    }

}
