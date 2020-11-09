pipeline {
    agent any

    tools {
        nodejs "node"
        maven "M3"
    }

    environment { 
        registry = "agusvelez5/spring-boot" 
        registryCredential = 'docker-hub-credentials' 
    }
    
    stages {
        stage('Build') {
            steps {
                git(url: 'https://github.com/AgusVelez5/spring-boot-app.git', branch: 'master', poll: true)
                sh 'docker build -t $registry:$BUILD_ID .'
            }
        }
        
        stage('Prepare tests') {
            steps{
                sh script:'''
                  #!/bin/bash
                  mvn -Dmaven.test.failure.ignore=true clean package
                  cd ./it
                  npm install
                '''
            }
        }
        
        stage('Tests') {
            steps {
                sh script:'''
                  #!/bin/bash
                  java -jar target/spring-boot-sample-actuator-2.0.2.jar &
                  sleep 7
                  cd ./it
                  npx codeceptjs run --steps --reporter mocha-multi
                '''
            }
            
            post {
                success {
                    junit '**/output/*.xml'
                }
            }
        }
        
        stage('Static code analisis') {
            steps {
                sh 'mvn sonar:sonar -Dsonar.projectKey=AgusVelez5_spring-boot-app -Dsonar.organization=agusvelez5 -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=b378dbfb4258b2712dd3dc78052400611207bd8a  -Dmaven.test.failure.ignore=true -Dsonar.scanner.force-deprecated-java-version=truenode'
            }
        }
        
        stage('Push image') {
            steps {
                withCredentials([usernamePassword( credentialsId: 'docker-hub-credentials', usernameVariable: 'USERNAME', passwordVariable: 'PASSWORD')]) {
                    script {
                        docker.withRegistry('', 'docker-hub-credentials') {
                            sh "docker login -u ${USERNAME} -p ${PASSWORD}"
                            sh "docker push $registry:$BUILD_ID"
                        }   
                    }
                }   
            }
        }
    }
}
