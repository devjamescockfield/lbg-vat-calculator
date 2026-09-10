pipeline { 
    agent any 
    environment { 
        dockerCreds = credentials('dockerhub_login') 
        registry = "${dockerCreds_USR}/vatcal" 
        registryCredentials = "dockerhub_login" 
        dockerImage = "" // empty var, will be written to later 
    } 
    
    stages { 
        stage('Run Tests') { 
            steps { 
                sh 'npm install' 
                sh 'CI=true npm test' 
            } 
        } 
        
        stage('Build Image') { 
            steps { 
                script { 
                    dockerImage = docker.build(registry) 
                } 
            }
        } 

        stage('Analyze Image') {
            steps {
                script {
                    sh """
                        docker run --rm \
                            -v /var/run/docker.sock:/var/run/docker.sock \
                            -v \$(pwd)/.dive-ci.yml:/.dive-ci.yml \
                            wagoodman/dive:latest ${registry}:${env.BUILD_NUMBER} \
                            --ci --ci-config /.dive-ci.yml
                    """
                }
            }
        }
        
        stage('Push Image') { 
            steps { 
                script { 
                    docker.withRegistry("", registryCredentials) { 
                        dockerImage.push("${env.BUILD_NUMBER}") 
                        dockerImage.push("latest") 
                    } 
                } 
            } 
        } 
        
        stage('Clean Up') { 
            steps { 
                sh "docker image prune --all --force --filter 'until=48h'" 
            } 
        } 
    } 
}
