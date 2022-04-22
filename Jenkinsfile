pipeline{
    agent any
    environment { 
        registry = "furqano/restapi-app" 
        registryCredential = 'dockhub' 
        dockerImage = '' 
    }
    stages{
      stage('Clone Repo') {
      agent { label 'docker' }
        steps {
            git branch: '**', url: 'https://github.com/furqano/Java-Rest-API-2-Server.git'
            
            }
        }
      stage('Build app') {
      agent { label 'docker' }
        steps {
            withMaven(maven :'maven'){
                sh "mvn package"
            }
            
            }
        }
      stage('Build Image'){
            agent { label 'docker' }
            steps{
                
            
            script{
                dockerImage = docker.build("furqano/test")
            }
        }}
        stage('Deploy our image') { 
            agent { label 'docker' }
            steps { 
                script { 
                    docker.withRegistry('https://registry.hub.docker.com', 'dockhub') {            
       dockerImage.push("latest")        
              }    
                   
                    }
                } 
            }
            stage("Helm deploy"){
                agent{ label 'helm'}
                steps{
                    
                
                script{
                    bat "helm upgrade dep2 -n dep test"
                }}
            }
        } 
    }