pipeline {
    agent any
    
    tools{
        maven 'localMaven'
        jdk 'localJdk'
    }

    environment {       
        WORKSPACE = "${env.WORKSPACE}"
    }

    stages {
        stage('Git checkout') {
            steps {
                echo 'Cloning the application codebase'
                git branch: 'main', url: 'https://github.com/kenechikwendu/Jenkins-CI-CD-Pipeline-Project.git'
                
            }
        }
        stage('Build') {
            steps {
                sh 'mvn clean package'
                sh 'java --version'
                
            }
            post{
                success{
                    echo 'archiving'
                    archiveArtifacts artifacts: '**/*.war', followSymlinks: false
                }
            }
        }
        stage('Unit Test'){
            steps {
              sh 'mvn test'
            }
        }
        stage('Integration Test'){
            steps {
              sh 'mvn verify -DskipUnitTests'
            }
        }
        stage ('Checkstyle Code Analysis'){
            steps {
              sh 'mvn checkstyle:checkstyle'
            }
            post {
              success {
                echo 'Generated Analysis Result'
              }
            }
        }
        stage('SonarQube scanning') {
            steps {
             withSonarQubeEnv('SonarQube'){
             sh """ mvn sonar:sonar \
               -Dsonar.projectKey=JavaWebApp \
               -Dsonar.host.url=http://172.31.20.66:9000 \
               -Dsonar.login=a69528e7a0330b18b07f5d5416b783c3cb1dabb3 """
             }
            }
        }
        stage('Quality Gate'){
            steps{
                waitForQualityGate abortPipeline: true 
            }
        }
        stage('Upload Artifact to Nexus'){
            steps{
                sh 'mvn clean deploy -DskipTests'
            }
        }

        stage('Deploy to DEV') {
          environment {
            HOSTS = "dev"
          }
          steps {
            sh "ansible-playbook ${WORKSPACE}/deploy.yaml --extra-vars \"hosts=$HOSTS workspace_path=$WORKSPACE\""
          }
        }
               
    }
}
