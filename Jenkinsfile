pipeline{
   agent any
    stages{
        // stage("Cleanup") {
        //     steps {
        //         cleanWs() // Move cleanWs() inside the steps block in a stage
        //     }
       // }
        stage("w/o docker"){
            steps{
                sh '''
                echo "executing without docker"
                ls -la 
                touch docker-no.txt
                '''
            }
            
        }

        /* stage("Build"){
            agent {
                docker {
                 image 'node:latest'
                 reuseNode true
                }
            }
            steps{
                sh '''
                node --version 
                npm --version
                npm ci
                npm run build
                ls -la
                '''
            }


            
        } */

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
                    image 'mcr.microsoft.com/playwright:v1.39.0-jammy'
                    reuseNode true
                }
            }

                steps{
                    sh '''
                    npm install  serve
                    node_modules/.bin/serve -s build &
                    sleep 30
                    npx playwright test --reporter=html
                    '''
                }
        }

        stage("deploy"){
            agent {
                docker {
                    image 'node:latest'
                    reuseNode true
                }

            }
            steps {
                npm install netlify-cli
                node_modules/.bin/netlify --version
            }
        }
    }

    post {
        always {
            junit 'jest-results/junit.xml'
            publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: 'playwright-report', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true])
        }
    }
    
}