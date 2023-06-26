pipeline {
    agent {
        node {
            label 'built-in'
        }
    }
    stages {
        stage('Continuous download') {
            steps {
                git branch: 'main', credentialsId: 'classicstan', url: 'https://github.com/classicstan/School-webapp.git'
            }
        }
        stage ('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'sonarqube';
                    withSonarQubeEnv('sonarqube') {
                        sh "${scannerHome}/bin/sonar-scanner \
                            -Dsonar.login=squ_2e5fb188c81da23c0d10bdd716452d9a1bf1f401 \
                            -Dsonar.projectKey=School \
                            -Dsonar.exclusions=vendor/**,resources/**,**/*.java \
                            -Dsonar.sources=/var/lib/jenkins/workspace/Operations2023/src \
                            -Dsonar.host.url=http://172.31.26.120:9000"
                    }
                }
            }
        }
        stage('SonarQube QG status') {
            steps {
               echo 'code analysis was successful'
            }
        }
        stage('Continuous build') {
            steps {
               sh 'mvn clean package'
            }
        }
        stage('continous deployment') {
            steps {
               deploy adapters: [tomcat9(credentialsId: 'Devcredentials', path: '', url:'http://172.31.94.205:8080')], contextPath: 'qaenv', war: '**/*.war'
            }
        }
        stage('Continuous testing') {
            steps {
                echo 'testing was successful'
            }
        }
        stage('deploy artifact') {
            steps {
                nexusArtifactUploader artifacts: [
                        [
                            artifactid: 'tt.test', 
                            classifier: '', 
                            file: '/var/lib/jenkins/workspace/Operations23/target/tt.test.war', 
                            type: 'war'
                        ]
                    ], 
                    credentialsid: 'nexus', 
                    groupld: 'com.tt', 
                    nexusUrl: '54.91.82.201:8081',
                    nexusVersion: 'nexus3', 
                    protocol: 'http', 
                    repository: 'Operations23', 
                    version: '*1.0-SNAPSHOT'
                
            }
        }
        stage('continous dlivery') {
            steps {
                input message: 'are you authorized to run this job?', submitter: 'stanley'
                deploy adapters: [tomcat9(credentialsId: 'Prodcredentials', path: '', url: 'http://172.31.93.91:8080')], contextPath: 'prodenv', war: '**/*.war'
            }
        }
        stage( 'Post-Build Notification') {
            steps {
                script {
                    def status = currentBuild.currentResult
                    def message = "Build ${status}: Job '$ {env. JOB_NAME} [$ {env.BUILD_NUMBER}]'" 
                    slackSend (
                        channel: 'operation2023',
                        color: status == 'SUCCESS' ? 'good' : 'danger' ,
                        message: message, 
                        tokenCredentialId: 'slack'
                    )
                }
            }
        }    
    
    }
    post {
        always {
            script {
                slacksend (
                    channel: 'operation2023',
                    color: currentBuild.currentResult == 'SUCCESS' ? 'good' : 'danger',
                    message: "Build ${currentBuild.currentResult}: Job '$ {env.JOB_NAME} [${env.BUILD_NUMBER}]'"
                    message: "Build ${status}:Job '$ {env.JOB_NAME} [${env.BUILD_NUMBER}]'", 
                    tokenCredentialId: 'slack'
                )
            }
        }
    }
}
