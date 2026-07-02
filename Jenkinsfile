pipeline {

    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"      // Adds Maven to system PATH
    }

    stages {

        stage("Checkout") {
            steps {
                echo "----------- Checkout Started ----------"
                checkout scm               // Pulls source code from configured SCM (Git)
                echo "----------- Checkout Completed ----------"
            }
        }

        stage("Build") {
            steps {
                echo "----------- Build Started ----------"
                sh 'mvn clean install'     // Cleans previous builds and installs the package
                echo "----------- Build Completed ----------"
            }
        }

    }

}
