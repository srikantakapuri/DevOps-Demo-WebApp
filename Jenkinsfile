pipeline {
    agent any
    tools {
        maven 'Maven3.6.3'
        
    }
    stages {
        stage('Checkout') {
            steps {
                checkout([$class: 'GitSCM', branches: [[name: '*/master']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'https://github.com/Sathish-1985/DevOps-Demo-WebApp']]])
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean install -f pom.xml'
        }
    }
      
       stage('SonarqubeScanner') {
                environment {
                    scannerHome = tool 'sonarqube'
                }
                steps {
                   //slackSend channel: 'alerts', message: 'Static code analysis is in progress'
                    withSonarQubeEnv('sonarqube') {
                        sh """${scannerHome}/bin/sonar-scanner"""
                    }
                   // timeout(time: 2, unit: 'MINUTES') {
                   //     waitForQualityGate abortPipeline: true
                    //}
                    //slackSend channel: 'alerts', message: 'Static code analysis is complete'
                    }
            }
            	stage("artifactory upload"){
    	    steps{
    	        rtUpload (
                    serverId: 'artifactory',
                    spec: '''{
                          "files": [
                            {
                              "pattern": "**/*.war",
                              "target": "libs-release-local"
                            }
                         ]
                    }''',
                    buildName: 'holyFrog',
                    buildNumber: '42'
                )
    	    }
    	}
        stage('Test Deploy ') {
            steps {
                    deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://18.224.107.247:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
                
        }
    }
        stage('Slack Notification after Test') {
            steps {
                  slackSend channel: '#alert', message: 'Deploy to Test Success'  
                
        }
    }
        stage('Jira Update after deploy on Test') {
            
            steps {
                jiraSendBuildInfo branch: 'BUG-2', site: 'sathishdevops.atlassian.net'
        }
    }
    	stage("unit test"){
    		steps{
    		   sh "mvn -B -f /var/lib/jenkins/workspace/Test/functionaltest/pom.xml install"
    		}
            post {
                success {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])
                }
            }
    	}
        stage('Deploy to Prod') {
            steps {
                 deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://3.16.37.146:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
        }
    }
          stage('Jira Update after deploy on Prod') {
            
            steps {
                jiraSendDeploymentInfo environmentId: 'http://3.16.37.146:8080/', environmentName: 'http://3.16.37.146:8080/', environmentType: 'testing', issueKeys: ['BUG-2'], serviceIds: [''], site: 'sathishdevops.atlassian.net', state: 'successful'
        }
    }
    
        stage('Slack Notification after Prod ') {
            steps {
                  slackSend channel: '#alert', message: 'Deploy to Prod Success'  
                
        }
    }
stage("Sanity test"){
    		steps{
    		   sh "mvn -B -f /var/lib/jenkins/workspace/Test/Acceptancetest/pom.xml install"
    		}
            post {
                success {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
                }
            }
    	}
 }
}
