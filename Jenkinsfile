pipeline {
    agent any
    environment {
        PATH = "/opt/maven/bin:$PATH"
    }
    stages {
        stage( MyBuild ) {
            steps {
                sh 'mvn clean install'
            }
        }
    }
}
