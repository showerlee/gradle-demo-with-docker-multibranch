#!groovy

pipeline {
    agent none

    options {
        timestamps ()
        ansiColor('xterm')
    }
    
    environment {
        PATH="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/root/bin"
        PROJECT_NAME="gradle-demo-with-docker"
        GITHUB_CREDENTIAL_ID="Github-credential"
        NEXUS_URL="nexus.example.com:8082"
        NEXUS_CREDENTIAL_ID="Nexus-credential"
        SSH_USER="root"
        SSH_PORT="22"
        APP_PORT="8083"
    }
    
    stages {
        stage("Functional test"){
            agent{
                node {
                    label 'master'
                }
            }
            steps{
                sh """
                ./auto/run-functional-test
                """
            }
        }

        stage("Build and Release Image"){
            agent{
                node {
                    label 'master'
                }
            }
            when {
                branch 'master'
            }
            steps{
                withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDENTIAL_ID}", usernameVariable: 'Nexus_USERNAME', passwordVariable: 'Nexus_PASSWORD')]) {
                    sh """
                    ./auto/login-nexus ${env.Nexus_USERNAME} ${env.Nexus_PASSWORD} ${env.NEXUS_URL}
                    ./auto/release-image
                    """
                }
                echo "[INFO] Upload image version"
                script {
                    IMAGE=readFile('artifacts/docker-image-released.txt').trim()
                }
            }
        }

        stage("Initialize test env"){
            environment {
                DEPLOY_ENV="test"
            }
            agent{
                node {
                    label 'master'
                }
            }
            when {
                branch 'master'
            }
            steps{
                sh """
                ./auto/clean-resource
                ./auto/test-ssh-connection ${env.SSH_USER} ${env.DEPLOY_ENV} ${env.SSH_PORT}
                ./auto/check-resource ${env.SSH_USER} ${env.DEPLOY_ENV} ${env.SSH_PORT}
                """
            }
        }

        stage("Deploy test env via ansible"){
            environment {
                DEPLOY_ENV="test"
            }
            agent{
                node {
                    label 'master'
                }
            }
            when {
                branch 'master'
            }
            steps{
                echo "[INFO] Download image version"
                sh """
                mkdir artifacts/
                echo ${IMAGE} > artifacts/docker-image-released.txt
                """
                withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDENTIAL_ID}", usernameVariable: 'Nexus_USERNAME', passwordVariable: 'Nexus_PASSWORD')]) {
                    sh "auto/ansible-playbook ${env.PROJECT_NAME} ${env.DEPLOY_ENV} ${env.Nexus_USERNAME} ${env.Nexus_PASSWORD} ${env.NEXUS_URL}"
                }
            }
        }

        stage("Health Check in test env"){
            environment {
                DEPLOY_ENV="test"
            }
            agent{
                node {
                    label 'master'
                }
            }
            when {
                branch 'master'
            }
            steps{
                sh "./auto/health-check ${env.DEPLOY_ENV} ${env.APP_PORT} ${env.PROJECT_NAME}"
                input("Start to deploy in prod?")
            }
        }

        stage("Initialize prod env"){
            environment {
                DEPLOY_ENV="prod"
            }
            agent{
                node {
                    label 'master'
                }
            }
            when {
                branch 'master'
            }
            steps{
                sh """
                ./auto/clean-resource
                ./auto/test-ssh-connection ${env.SSH_USER} ${env.DEPLOY_ENV} ${env.SSH_PORT}
                ./auto/check-resource ${env.SSH_USER} ${env.DEPLOY_ENV} ${env.SSH_PORT}
                """
            }
        }

        stage("Deploy prod env via ansible"){
            environment {
                DEPLOY_ENV="prod"
            }
            agent{
                node {
                    label 'master'
                }
            }
            when {
                branch 'master'
            }
            steps{
                echo "[INFO] Download image version"
                sh """
                mkdir artifacts/
                echo ${IMAGE} > artifacts/docker-image-released.txt
                """
                withCredentials([usernamePassword(credentialsId: "${env.NEXUS_CREDENTIAL_ID}", usernameVariable: 'Nexus_USERNAME', passwordVariable: 'Nexus_PASSWORD')]) {
                    sh "auto/ansible-playbook ${env.PROJECT_NAME} ${env.DEPLOY_ENV} ${env.Nexus_USERNAME} ${env.Nexus_PASSWORD} ${env.NEXUS_URL}"
                }
            }
        }

        stage("Health Check in prod env"){
            environment {
                DEPLOY_ENV="prod"
            }
            agent{
                node {
                    label 'master'
                }
            }
            when {
                branch 'master'
            }
            steps{
                sh "./auto/health-check ${env.DEPLOY_ENV} ${env.APP_PORT} ${env.PROJECT_NAME}"
            }
        }

    }

}
