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

        stage('Analyze Image') {
            steps {
                script {
                    sh '''
                        if ! command -v dive &> /dev/null; then
                            wget https://github.com/wagoodman/dive/releases/download/v0.13.1/dive_0.13.1_linux_amd64.deb
                            apt install ./dive_0.12.0_linux_amd64.deb -y || dpkg -i ./dive_0.12.0_linux_amd64.deb
                        fi
                    '''
                    sh "CI=true dive ${registry}:${env.BUILD_NUMBER} --ci-config .dive-ci.yml"
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
