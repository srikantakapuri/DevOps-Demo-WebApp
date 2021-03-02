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
        stage('Build') {
            steps {
               
                sh 'mvn clean install -f pom.xml'
                 slackSend channel: '#alert', message: 'Build process completed' 
        }
        post {
       always {
           jiraSendBuildInfo branch: 'BUG-2', site: 'sathishdevops.atlassian.net'
           }
            }
    }
 
    stage('DeployToNewTest') {
            steps {
                
           deploy adapters: [tomcat8(url: 'http://18.216.215.135:8080/', credentialsId: 'tomcat', path: '' )], contextPath: '/QAWebapp', war: '**/*.war'
            slackSend channel: '#alert', message: 'Deploy to Test Success' 
        }
                post {
       always {
           jiraSendDeploymentInfo environmentId: 'http://18.216.215.135:8080/', environmentName: 'http://18.216.215.135:8080/', environmentType: 'testing', issueKeys: ['BUG-2'], serviceIds: [''], site: 'sathishdevops.atlassian.net', state: 'successful'
           jiraIssueSelector(issueSelector: [$class: 'ExplicitIssueSelector', issueKeys: 'BUG-2'])
       }
            }
       }
 stage ('parallel') {
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
    	    }
    	}
    	stage("unit test"){
    		steps{
    		   sh "mvn -B -f /var/lib/jenkins/workspace/DeclarativePipelineJob/functionaltest/pom.xml install"
    		}
            post {
                success {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test', reportTitles: ''])
                }
            }
    	}
        
    }
 }
    	stage("deploy to Prod"){
    		steps{
    		   deploy adapters: [tomcat8(credentialsId: 'tomcat', path: '', url: 'http://18.189.26.183:8080/')], contextPath: '/ProdWebapp', war: '**/*.war'
    	          slackSend channel: '#alert', message: 'Prod deployment completed' 
    		}
    		                post {
       always {
           jiraSendDeploymentInfo environmentId: 'http://18.189.26.183:8080/', environmentName: 'http://18.189.26.183:8080/', environmentType: 'testing', issueKeys: ['BUG-2'], serviceIds: [''], site: 'sathishdevops.atlassian.net', state: 'successful'
           jiraIssueSelector(issueSelector: [$class: 'ExplicitIssueSelector', issueKeys: 'BUG-2'])
       }
            }
    		
    	}
    	stage("Sanity test"){
    		steps{
    		   sh "mvn -B -f /var/lib/jenkins/workspace/DeclarativePipelineJob/Acceptancetest/pom.xml install"
    		}
            post {
                success {
                   publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
                }
            }
    	}
 }
}
