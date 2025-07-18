pipeline{
    agent { label 'AGENT-1' } //jenkins node name
    environment{
        PROJECT = 'expense'     //We can use these as global variables
        COMPONENT = 'frontend' 
        appVersion = ''
        ACC_ID = '314146304089'
    }
    options{
        disableConcurrentBuilds() 
        timeout(time: 30, unit: 'MINUTES')
    }
     parameters{
        booleanParam(name: 'deploy', defaultValue: false, description: 'Toggle this value')
    }
    stages{
        stage('Read Version'){
            steps{
                script{
                    def packageJson = readJSON file: 'package.json'
                    appVersion = packageJson.version
                    echo "Version is: $appVersion"
                }
            }
        }
        stage('Docker build'){
            steps{
                script{
                    withAWS(region: 'us-east-1', credentials: 'aws-creds') {
                        sh """
                        aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com

                        docker build -t ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion} .

                        docker push ${ACC_ID}.dkr.ecr.us-east-1.amazonaws.com/${project}/${component}:${appVersion}
                        """
                    }     
                }
            }
        }
        stage('Trigger Deploy'){
            when{
                expression { params.deploy }
            }
            steps{
                    build job: 'frontend-cd', parameters: [string (name: 'version', value: "${appVersion}")], wait: true
            }
        }
    } 
    post{
        always{
            echo 'I will always say Hello again!'
            deleteDir() //it will deletes the directory
        }
        failure{
            echo 'I will run when pipeline is failed'
        }
        success{
            echo 'I will run when pipeline is success'
        }
    }
}