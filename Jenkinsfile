pipeline {
    agent any

    stages {

    stage('Compile') {
    steps {
        bat 'mvn clean compile'
        }
    }
    stage('Unit Test') {
    steps {
        bat 'mvn test'
    }
    }
    stage('SonarQube Analysis') {
    steps {
        withSonarQubeEnv('SonarQube') {
            bat 'mvn sonar:sonar'
        }
    }
    }
    stage('Package') {
    steps {
        bat 'mvn clean package -DskipTests'
    }
    }
    stage('Build Docker Image') {
    steps {
        bat 'docker build -t springboot-demo:1.0 .'
    }
    }
    stage('Push Image') {
    steps {
        bat '''
        docker tag springboot-demo:1.0 localhost:8083/springboot-demo:1.0
        docker push localhost:8083/springboot-demo:1.0
        '''
    }
    }
    stage('Deploy') {
    steps {
        bat '''
        docker rm -f springboot-demo || exit 0
        docker run -d ^
            --name springboot-demo ^
            -p 8080:8080 ^
            localhost:8083/springboot-demo:1.0
        '''
    }
    }

    stage('Verify Deployment') {
    steps {
        bat 'docker ps'
    }
    }
}