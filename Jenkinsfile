pipeline {
    agent any
    tools { 
        maven 'Maven' 
        jdk 'jdk' 
    }
    stages {
        stage('Git Clone') {
            steps {
                git 'https://github.com/srikantakapuri/DevOps-Demo-WebApp.git'
            }
        }
        stage('Code compile') {
            steps {
               sh "mvn compiler:compile"
            }
        }
        stage('SonarqubeScanner') {
                environment {
                    scannerHome = tool 'sonarqube'
                }
                steps {
                   
                    withSonarQubeEnv('sonarqube') {
                        sh """${scannerHome}/bin/sonar-scanner"""
                    }
                   
                    }
            }
        stage('Code Package') {
            steps {
               sh "mvn clean install"
            }
            post {
               always {
                   jiraSendBuildInfo branch: 'BUG-2', site: 'sathishdevops.atlassian.net'
                   }
            }
        }
        stage ('artifactory & Dockerhub upload') {
    parallel {
       stage("artifactory upload"){
    	    steps{
    		      
                rtMavenDeployer (
                    id: 'deployer-unique-id',
                    serverId: 'artifactory',
                    releaseRepo: 'libs-release-local',
                    snapshotRepo: 'libs-snapshot-local'
                )
                rtMavenRun (
                    pom: 'pom.xml',
                    goals: 'clean install',
                    deployerId: 'deployer-unique-id'
                )
    	       rtPublishBuildInfo (
                    serverId: "artifactory"
                )
                slackSend channel: '#alerts', message: 'Artifactory upload completed' 
    	    }
    	}
    	stage('Push test image to DockerHub') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'Test-env', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook -i /etc/ansible/hosts /home/ansadmin/Devops_case_study/create-simple-docker-project.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//home//ansadmin//Devops_case_study', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            
                slackSend channel: '#alerts', message: 'Test Image is  pushed to Dockerhub' 
            }
            
        }
    	
               
    }
 }
        
        stage('Deploying in Test from Dockerhub') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'Test-env', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook -i /etc/ansible/hosts /home/ansadmin/Devops_case_study/simple-docker-project.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: false)])
            
                slackSend channel: '#alerts', message: 'Test deployment completed' 
            }
            post {
                always {
                    jiraSendDeploymentInfo environmentId: 'http://54.236.17.79:8080/', environmentName: 'http://54.236.17.79:8080/', environmentType: 'testing', issueKeys: ['BUG-2'], serviceIds: [''], site: 'sathishdevops.atlassian.net', state: 'successful'
                }
            }
        }

        stage("UI Test"){
    		steps{
    		   sh "mvn -B -f /var/lib/jenkins/workspace/Test-Project/functionaltest/pom.xml install"
    		}
            post {
                success {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])
                }
            }
    	}
        stage('Push prod image to DockerHub ') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'Test-env', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook -i /etc/ansible/hosts /home/ansadmin/Devops_case_study/prod/create-simple-docker-project.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '//home//ansadmin//Devops_case_study//prod', remoteDirectorySDF: false, removePrefix: 'target', sourceFiles: 'target/*.war')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
            
                slackSend channel: '#alerts', message: 'Prod Image is  pushed to Dockerhub' 
            }
        }
        stage('Deploying in Prod from Dockerhub') {
            steps {
                sshPublisher(publishers: [sshPublisherDesc(configName: 'Test-env', transfers: [sshTransfer(cleanRemote: false, excludes: '', execCommand: 'ansible-playbook -i /etc/ansible/hosts /home/ansadmin/Devops_case_study/prod/simple-docker-project.yml', execTimeout: 120000, flatten: false, makeEmptyDirs: false, noDefaultExcludes: false, patternSeparator: '[, ]+', remoteDirectory: '', remoteDirectorySDF: false, removePrefix: '', sourceFiles: '')], usePromotionTimestamp: false, useWorkspaceInPromotion: false, verbose: true)])
            
                slackSend channel: '#alerts', message: 'Prod deployment completed' 
            }
            post {
               always {
                   jiraSendDeploymentInfo environmentId: 'http://54.165.166.19:8080/', environmentName: 'http://54.165.166.19:8080/', environmentType: 'production', issueKeys: ['BUG-2'], serviceIds: [''], site: 'sathishdevops.atlassian.net', state: 'successful'
               }
            }
        }
        stage("Sanity test"){
    		steps{
    		   sh "mvn -B -f /var/lib/jenkins/workspace/Test-Project/Acceptancetest/pom.xml install"
    		}
    		post {
                success {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
                }
            }
    	}
    }
    
}
