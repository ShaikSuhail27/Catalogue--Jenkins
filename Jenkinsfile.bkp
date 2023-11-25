pipeline {
    agent { node { label 'Agent-1' } }
    //since variable declared here in environemnt there is no need of def

    environment{
        packageVersion = ''
        
    }
    stages {
        stage('package version') {
            steps {
                script {
                    def packageJson = readJSON(file: 'package.json')
                    packageVersion = packageJson.version
                    echo "version: ${packageVersion}"
                }
            }
        }
        stage('Install depdencies') {
            steps {
                sh 'npm install'
            }
        }
        stage('Unit test') {
            steps {
                echo "unit testing is done here"
            }
        }
        //sonar-scanner command expect sonar-project.properties should be available
        // stage('Sonar Scan') {
        //     steps {
        //         sh 'ls -ltr'
        //         sh 'sonar-scanner'
        //     }
        // }
         stage('Sonar Scan') {
            steps {
                echo "Sonar Scan is done here"
            }
        }
        stage('Build') {
            steps {
                sh 'ls -ltr'
                sh 'zip -r catalogue.zip ./* --exclude=.git --exclude=.zip'
            }
        }

         stage('SAST') {
            steps {
                echo "SAST is done here"
                echo "Version is ${packageVersion}"
            }
        }
       //pipeline utility steps plugin used to install for getting version
       // nexus artifact uplaoder plugin need to install
        stage('Publish Artifact') {
            steps {
                nexusArtifactUploader(
                    nexusVersion: 'nexus3',
                    protocol: 'http',
                    nexusUrl: '172.31.95.215:8081/',
                    groupId: 'com.roboshop',
                    version: "${packageVersion}",
                    repository: 'catalogue',
                    credentialsId: 'nexus-auth',
                    artifacts: [
                        [artifactId: 'catalogue',
                        classifier: '',
                        file: 'catalogue.zip',
                        type: 'zip']
                    ]
                )
            }
        }

       // to call the downstream job(deployment) we need to call it here  and we need to pass package version for deployment
        stage('Deploy') {
            steps {
                script {
                    echo "Deployment"
                def params =[
                    string(name:'version',value:"$packageVersion")
                ]
                
                 build job: "../catalogue-deploy/", wait: true,parameters: params
                }
            }
        }
    }

    post{
        always{
            echo 'cleaning up workspace'
            deleteDir()
        }
    }
}