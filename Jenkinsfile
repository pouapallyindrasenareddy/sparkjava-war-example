pipeline {
    agent any
    environment {
        PATH = "/opt/apache-maven-3.9.16/bin:$PATH"
    }
    stages {
        stage( MyBuild ) {
            steps {
                sh 'mvn clean install'
            }
        }
    }
}
