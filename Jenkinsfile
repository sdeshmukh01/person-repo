#!/usr/bin/env groovy

pipeline {
    agent any

    triggers {
        pollSCM('*/15 * * * *')
    }

    options { disableConcurrentBuilds() }

    stages {
        stage('Permissions') {
            steps {
                sh 'chmod 775 *'
            }
        }
		
		stage('Cleanup') {
            steps {
                sh './gradlew --no-daemon clean'
            }
        }

//        stage('Check Style, FindBugs, PMD') {
//            steps {
//                sh './gradlew --no-daemon checkstyleMain checkstyleTest findbugsMain findbugsTest pmdMain pmdTest cpdCheck'
//            }
//			post {
//				always {
//					step([
//						$class         : 'FindBugsPublisher',
//						pattern        : 'build/reports/findbugs/*.xml',
//						canRunOnFailed : true
//					])
//					step([
//						$class         : 'PmdPublisher',
//						pattern        : 'build/reports/pmd/*.xml',
//						canRunOnFailed : true
//					])
//					step([
//						$class: 'CheckStylePublisher',
//						pattern: 'build/reports/checkstyle/*.xml',
//						canRunOnFailed : true
//					])
//				}
//			}
//        }
		
		stage('Test') {
            steps {
                sh './gradlew --no-daemon check'
            }
            post {
                always {
                    junit 'build/test-results/test/*.xml'
                }
            }
        }

        stage('Build') {
            steps {
                sh './gradlew --no-daemon build'
            }
        }

        stage('Update Docker UAT image') {
//            when { branch "master" }
            steps {
                sh '''
					docker login -u "sagdeshm1" -p "Happy@123"
                    docker build --no-cache -t person .
                    docker tag person:latest sagdeshm1/person:latest
                    docker push sagdeshm1/person:latest
		                                docker.withRegistry('https://061369422772.dkr.ecr.ap-south-1.amazonaws.com', 'ecr:ap-south-1:AKIAQ4SPHCO2BNGETROM') {
            docker.image('person').push('latest')
					docker rmi person:latest
                '''
            }
        }

        stage('Update UAT container') {
//            when { branch "master" }
            steps {
                sh '''
					docker login -u "sagdeshm1" -p "Happy@123"
                    docker pull sagdeshm1/person:latest
					docker stop person
					docker rm person
					docker run -p 9090:9090 --name person -t -d sagdeshm1/person
					docker rmi -f $(docker images -q --filter dangling=true)
                '''
            }
        }

        stage('Release Docker image') {
            when { buildingTag() }
            steps {
                sh '''
					docker login -u "sagdeshm1" -p "Happy@123"
                    docker build --no-cache -t person .
                    docker tag person:latest sagdeshm1/person:${TAG_NAME}
                    docker push sagdeshm1/person:${TAG_NAME}
					docker rmi $(docker images -f “dangling=true” -q)
               '''
            }
        }
    }
}
