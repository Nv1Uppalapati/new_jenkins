pipeline {
    agent { label 'master'}

    environment {
        function_name = 'java2121'
    }

    stages {

         //CI Start
        stage('Build') {
            steps {
                echo 'Build'
                sh 'mvn package'
            }
        }


         stage("SonarQube analysis") {
             agent { label 'jenkins-slave' }


             when {
                 anyOf {
                     branch 'feature/*'
                     branch 'main'
                 }
             }
             steps {
                 withSonarQubeEnv('Sonar') {
                     sh 'mvn sonar:nv1uppalapati'
                 }
             }
         }

         stage("Quality Gate") {
             steps {
                 script {
                     try {
                         timeout(time: 10, unit: 'MINUTES') {
                             waitForQualityGate abortPipeline: true
                         }
                     }
                     catch (Exception ex) {

                     }
                 }
             }
         }

        stage('Push') {
            steps {
                echo 'Push'

                sh "aws s3 cp target/sample-1.0.3.jar s3://slave2121"
            }
        }

        // Ci Ended

        // CD Started

        stage('Deployments') {
            parallel {

                stage('Deploy to Dev') {
                    steps {
                        echo 'Build'

                        sh "aws lambda update-function-code --function-name $function_name --region us-east-2 --s3-bucket slave2121 --s3-key sample-1.0.3.jar"
                    }
                }

                stage('Deploy to test ') {
                    when {
                        branch 'main'
                    }
                    steps {
                        echo 'Build'

                         sh "aws lambda update-function-code --function-name $function_name --region us-east-2 --s3-bucket slave2121 --s3-key sample-1.0.3.jar"
                    }
                }
            }
        }

        stage('Deploy to Prod') {
            when {
                branch 'main'
            }
            steps {
               input (
                    message: 'Are we good for Prod Deployment ?'
               )
            }
        }

        stage('Release to Prod') {
            when {
                branch 'main'
            }
            steps {
                sh "aws lambda update-function-code --function-name $function_name --region us-east-2 --s3-bucket slave2121 --s3-key sample-1.0.3.jar"
            }
        }


        

        // CD Ended
    }

    post {
        always {
            echo "${env.BUILD_ID}"
            echo "${BRANCH_NAME}"
            echo "${BUILD_NUMBER}"

        }

        failure {
            echo 'failed'
        }
        aborted {
            echo 'aborted'
        }
    }
}
