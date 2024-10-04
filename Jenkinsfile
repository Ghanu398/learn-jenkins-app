pipeline{
   agent any
   environment {
    NETLIFY_SITE_ID = '3c27be64-111d-4f06-9df1-496a4ad97a54'
    NETLIFY_AUTH_TOKEN = credentials("netlify-token")
    REACT_APP_VERSION = "1.2.${BUILD_ID}"
    
   }
    stages{
        // stage("Cleanup") {
        //     steps {
        //         cleanWs() // Move cleanWs() inside the steps block in a stage
        //     }
       // }

       
        stage("docker build"){
            steps{
                sh '''
                docker image build -t manual .
                '''
            }
            
        }

        stage("Build"){
            agent {
                docker {
                 image 'node:latest'
                 reuseNode true
                }
            }
            steps{
                sh '''
                npm ci
                npm run build

                '''
            }


            
        }

        stage('aws') {
            agent {
                docker {
                    image 'amazon/aws-cli'
                    args "--entrypoint=''"
                    reuseNode true
                }
            }
            
            environment {
                AWS_S3_BUCKET = 'ghannu'
            }
            steps {
                withCredentials([usernamePassword(credentialsId: 'AWS', passwordVariable: 'AWS_SECRET_ACCESS_KEY', usernameVariable: 'AWS_ACCESS_KEY_ID')]) {
            sh '''
                aws --version
                aws s3 ls
                aws s3 sync build s3://$AWS_S3_BUCKET
                
                '''
}
               
            }
        } 

        stage("test"){
      agent{
        docker{
             image 'node:latest'
                 reuseNode true
        }
      }
        steps{
            sh '''
            ls -la
          # test -f build/index.html
            npm test
            '''
            
        }
        }

        stage("E2E"){
            agent {
                docker {
                    image 'manual'
                    reuseNode true
                }
            }

                steps{
                    sh '''
                    
                    serve -s build &
                    sleep 30
                    npx playwright test --reporter=html
                  
                    '''
                }

                post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
        }

        stage("dev deploy"){
            agent {
                docker {
                    image 'manual'
                    reuseNode true
                }

            }
             environment {
                    CI_ENVIRONMENT_URL = "testing"
                }
            steps {

                catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
               sh '''
                
                netlify --version
                # netlify link --id "$NETFLIFY_SITE_ID"
                netlify deploy --dir=build --json > deploy-output.json
                CI_ENVIRONMENT_URL=$(node-jq -r '.deploy_url' deploy-output.json)
                npx playwright test  --reporter=html
               '''

               
                }
            }

          post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'STAGING HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
        }

         

        // stage("Approval"){

            

        //     input {
        //         message 'Do you wish to deploy to production?'
        //         ok 'Yes, I am sure!'
        //     }

        //     steps {
        //         sh 'echo "approved and proceeding further for production deployment"'
        //     }
            
        // }

        stage("prod deploy"){
            agent {
                docker {
                    image 'manual'
                    reuseNode true
                }

            }
             environment {
                    CI_ENVIRONMENT_URL = 'https://ghanshyam123.netlify.app'
                }
            steps {
               sh '''
                
                # netlify link --id "$NETFLIFY_SITE_ID"
                netlify status
                netlify deploy --dir=build --prod
                npx playwright test --reporter=html
               '''
            }
                       post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'Prod HTML Report', reportTitles: '', useWrapperFileDirectly: true])
             }
                            }
        }
        
            
       

    }

    
    
}