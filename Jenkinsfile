#!groovy
import java.text.*

// pod utilisé pour la compilation du projet
podTemplate(label: 'books-api-pod', nodeSelector: 'medium', containers: [

        // le slave jenkins
        containerTemplate(name: 'jnlp', image: 'jenkinsci/jnlp-slave:alpine'),

        // un conteneur pour construire les images docker
        containerTemplate(name: 'docker', image: 'docker:18.09', command: 'cat', ttyEnabled: true)],

        // montage nécessaire pour que le conteneur docker fonction (Docker In Docker)
        volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
) {

    node('books-api-pod') {

        def branch = env.JOB_NAME.replaceFirst('.+/', '');

        properties([
                buildDiscarder(
                        logRotator(
                                artifactDaysToKeepStr: '1',
                                artifactNumToKeepStr: '1',
                                daysToKeepStr: '3',
                                numToKeepStr: '3'
                        )
                )
        ])

        def now = new SimpleDateFormat("yyyyMMddHHmmss").format(new Date())

        stage('CHECKOUT') {
            checkout scm
        }

        /* container('maven') {

            stage('BUILD SOURCES') {
                withCredentials([string(credentialsId: 'sonarqube_token', variable: 'token')]) {

                    // sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://sonarqube-sonarqube:9000 -Dsonar.java.binaries=target -Dsonar.login=${token} -DskipTests'
                     sh 'mvn clean package'
                }
            }
        } */

        container('docker') {

            stage('BUILD DOCKER IMAGE') {

                sh 'mkdir /etc/docker'

                // le registry est insecure (pas de https)
                sh 'echo {"insecure-registries" : ["registry.k8.wildwidewest.xyz"]} > /etc/docker/daemon.json'

                withCredentials([usernamePassword(credentialsId: 'nexus_user', usernameVariable: 'username', passwordVariable: 'password')]) {

                    sh "docker login -u ${username} -p ${password} registry.k8.wildwidewest.xyz"
                }

                sh "docker build . --tag registry.k8.wildwidewest.xyz/repository/docker-repository/opus/books-gui:$now"

                sh "docker push registry.k8.wildwidewest.xyz/repository/docker-repository/opus/books-gui:$now"
            }
        }

        stage('RUN') {

            build job: "/SofteamOuest-Opus/chart-run/master",
                    wait: false,
                    parameters: [string(name: 'image', value: "$now"),
                                 string(name: 'chart', value: "books-gui")]
        }

    }
}
