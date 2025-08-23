pipeline {
    agent any
    environment{
        SONAR_HOME = tool "sonar"
    }
    parameters {
        string(name: 'DOCKER_TAG', defaultValue: 'latest', description: 'Setting docker image for latest push')
    }
    stages{
        stage("clone code from github"){
            steps{
                git branch: 'DevOps', url: 'https://github.com/priyanshijat/Springboot-BankApp.git'
            }
        }
        stage("build an image"){
            steps{
                sh "docker build -t bank-app:${params.DOCKER_TAG} ."
            }
        }
        stage("Trivy: Filesystem scan"){
            steps{
                sh "trivy fs ."
            }
        }
        stage("OWASP Dependency-Check") {
            steps {
                dependencyCheck additionalArguments: '--scan ./ --format XML', odcInstallation: 'owasp'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }
        stage("SonarQube Code Quality Analysis") {
            steps {
                withSonarQubeEnv("sonar") {
                    sh '''
                    ${SONAR_HOME}/bin/sonar-scanner \
                    -Dsonar.projectKey=bankapp \
                    -Dsonar.projectName=bankapp \
                    -Dsonar.exclusions=**/*.java
                    '''
                }
            }
         }
         stage("SonarQube Quality Gate") {
             steps {
                 timeout(time: 2, unit: 'MINUTES') {
                     waitForQualityGate abortPipeline: true
                 }
              }
         }
        
        stage("push image on dockerhub"){
            steps{
                withCredentials([usernamePassword(
                    credentialsId: "dockerhubcreds",
                    usernameVariable: "dockerHubUser",
                    passwordVariable: "dockerHubPass"
                )]){
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPass}"
                    sh "docker image tag bank-app:${params.DOCKER_TAG} ${env.dockerHubUser}/bank-app:${params.DOCKER_TAG}"
                    sh "docker push ${env.dockerHubUser}/bank-app:${params.DOCKER_TAG}"
                }
            }
       }
    }
    post {
        success {
            script {
                emailext(
                    from: 'priyanshijat06@gmail.com',
                    to: 'priyanshijat06@gmail.com',
                    subject: 'Build Success for Bankapp CICD',
                    body: 'Build Success for Bankapp CICD'
                )
            }
        }
    }
}
