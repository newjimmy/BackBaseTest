#!/usr/bin/env groovy
import java.util.regex.Matcher
def validateParam(KEY, VAL) {
    if (VAL == null || VAL == '') {
        currentBuild.result = 'ABORTED'
        def err = KEY + " was not defined. Aborted."
        currentBuild.displayName = "#${BUILD_NUMBER} [ENV: ${KUBERNETES_ENVIRONMENT}] [REPO: ${REPOSITORY}] [BRANCH: ${BRANCH}] - ${err}"
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
        string(name: 'REPOSITORY', defaultValue: 'test', description: 'Repository name under corporate git like a https://bitbucket.com/projects/K8S')
        string(name: 'BRANCH', defaultValue: 'dev')
    }
    stages {
        stage("Validate params") {
            script {
                validateParam("REPOSITORY", params.REPOSITORY)
                validateParam("BRANCH", params.BRANCH)
            }
        }
        stage("Checkout repo"){
            steps{
                script {
                    checkout scm: [$class           : 'GitSCM',
                                   userRemoteConfigs: [[url: "https://bitbucket.com/projects/K8S/${params.REPOSITORY}.git", credentialsId: '65532f8a-77d0-4191-9e45-d10a2c37c772']],
                                   branches         : [[name: "${params.BRANCH}"]]], changelog: false, poll: false
                    sh "cd helm/flux && ls -la"
                }
            }
        }
        // Checking coding style
        stage('Lint') {
            steps {
                echo "Make some magic running \"lint\" and other tools to make sure what our code is clean so on "
                script {
                    withDockerContainer(image: 'private-docker-registry.org.com/java-lint', args: '-u root:root') {

                        sh '''
                        
                        env
                        '''
                    }

                }
            }
        }
        stage('Build') {
            steps {
                script {
                    withDockerContainer(image: 'private-docker-registry.org.com/java-mvn', args: '-u root:root') {

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
                    withDockerContainer(image: 'private-docker-registry.org.com/java-mvn', args: '-u root:root') {

                        sh '''
                        mvn test
                        '''
                    }

                }
            }
        }

    }
}
