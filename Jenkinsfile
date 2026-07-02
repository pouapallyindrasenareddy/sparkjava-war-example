pipeline {
    agent any

    environment {
        PATH = "/opt/maven/bin:$PATH"
        MAVEN_OPTS = "-Xmx1024m -XX:MaxPermSize=256m"   // JVM options for Maven
        APP_NAME = "my-app"                               // Application name
        VERSION = "1.0.${BUILD_NUMBER}"                   // Dynamic version using Jenkins build number
    }

    options {
        timestamps()                                      // Adds timestamps to console output
        timeout(time: 30, unit: 'MINUTES')                // Fails build if it exceeds 30 minutes
        buildDiscarder(logRotator(numToKeepStr: '10'))    // Keeps only the last 10 builds
        disableConcurrentBuilds()                         // Prevents concurrent builds
    }

    stages {

        stage("Checkout") {
            steps {
                echo "----------- Source Checkout Started ----------"
                checkout scm                              // Checks out source code from configured SCM
                echo "----------- Source Checkout Completed ----------"
            }
        }

        stage("Build") {
            steps {
                echo "----------- Build Started ----------"
                sh 'mvn clean install -DskipTests=true'   // Builds the project, skipping tests here
                echo "----------- Build Completed ----------"
            }
        }

        stage("Unit Test") {
            steps {
                echo "----------- Unit Tests Started ----------"
                sh 'mvn test'                             // Runs unit tests separately for clarity
                echo "----------- Unit Tests Completed ----------"
            }
            post {
                always {
                    junit '**/target/surefire-reports/*.xml'   // Publishes JUnit test reports
                }
            }
        }

        stage("Code Quality Check") {
            steps {
                echo "----------- SonarQube Analysis Started ----------"
                sh 'mvn sonar:sonar'                      // Runs SonarQube static code analysis
                echo "----------- SonarQube Analysis Completed ----------"
            }
        }

        stage("Package") {
            steps {
                echo "----------- Packaging Started ----------"
                sh 'mvn package -DskipTests=true'         // Creates the deployable artifact (JAR/WAR)
                archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
                                                          // Archives the built artifact in Jenkins
                echo "----------- Packaging Completed ----------"
            }
        }

        stage("Deploy to Dev") {
            when {
                branch 'develop'                          // Runs this stage only on the develop branch
            }
            steps {
                echo "----------- Deploying to Dev Environment ----------"
                sh 'mvn deploy -DskipTests=true -Pdev'   // Deploys to Dev using 'dev' Maven profile
                echo "----------- Deployment to Dev Completed ----------"
            }
        }

        stage("Deploy to Staging") {
            when {
                branch 'release/*'                        // Runs this stage only on release branches
            }
            steps {
                echo "----------- Deploying to Staging Environment ----------"
                sh 'mvn deploy -DskipTests=true -Pstaging'  // Deploys to Staging Maven profile
                echo "----------- Deployment to Staging Completed ----------"
            }
        }

        stage("Deploy to Production") {
            when {
                branch 'main'                             // Runs this stage only on the main branch
            }
            steps {
                input message: "Approve Deployment to Production?", ok: "Deploy"
                                                          // Manual approval gate before production deploy
                echo "----------- Deploying to Production ----------"
                sh 'mvn deploy -DskipTests=true -Pprod'  // Deploys to Production using 'prod' profile
                echo "----------- Production Deployment Completed ----------"
            }
        }
    }

    post {
        always {
            echo "----------- Pipeline Finished ----------"
            cleanWs()                                     // Cleans up workspace after every build
        }
        success {
            echo "========= BUILD SUCCEEDED ========="
            mail to: 'team@example.com',
                 subject: "SUCCESS: ${APP_NAME} Build #${BUILD_NUMBER}",
                 body: "Build ${VERSION} completed successfully.\nCheck: ${BUILD_URL}"
                                                          // Sends email notification on success
        }
        failure {
            echo "========= BUILD FAILED ========="
            mail to: 'team@example.com',
                 subject: "FAILURE: ${APP_NAME} Build #${BUILD_NUMBER}",
                 body: "Build ${VERSION} failed.\nCheck logs: ${BUILD_URL}console"
                                                          // Sends email notification on failure
        }
        unstable {
            echo "========= BUILD UNSTABLE (Test Failures) ========="
                                                          // Triggered when tests fail but build passes
        }
    }
}
