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

         stage("Build"){
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
            
            'test -f build/index.html'
            npm test
            '''
            
        }
        }
    }

    post {
        always {
            junit 'test-results/junit.xml'
        }
    }
    
}