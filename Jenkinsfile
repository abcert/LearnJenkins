pipeline {
    agent any
    stages {
        stage('Init'){
            steps {
                echo 'Initilizing Project and copying code from cloud repo'
            }
        }
        stage('Build'){
            steps {
                echo 'Building project in case of java or scala'
            }
        } 
        stage('Unit-Testing'){
            steps {
                echo 'Unit Testing is only possible in case unit test case written for java /scala'
            }
        }
        stage('Deploy'){
            steps{
                
                echo 'Deploy code to either dev/test/prod depends on pipleline status'
            }
        }
    }
}