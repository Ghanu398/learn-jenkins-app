pipeline{
   agent any
    stages{
        cleanWs()
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

        steps{
            echo "Test stage"
        }
        }
    }
    
}