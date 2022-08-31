pipeline {
    triggers {
       githubPush()
    }
    agent any
    tools {
          nodejs "Node"
          maven 'maven-3.6.3'
    }
    environment {
        CI = 'true'
        CURR_DIR = pwd()
        API_DIR = '/CRMAPI'
        DEV_ENV = 'dev'
        PROD_ENV = 'prod'
        TEST_SCRIPT_FILE = 'test_script.postman_collection_crmapi320.json' 
    }
    stages {
        
        
        stage('Preparation') {
            steps{
                sh '''
                    echo "PATH = ${PATH}"
                    echo "M2_HOME = ${M2_HOME}"
                '''
                git branch: "main",
                url: 'https://github.com/dushansachinda/apim_artifacts.git',
                credentialsId: 'demo-github-cred'
            }
        }
        

        stage('Deploy to Dev') {
            environment{
                RETRY = '80'
            }
            steps {
            
                echo 'Logging into $DEV_ENV'
                withCredentials([usernamePassword(credentialsId: 'dev_apim_admin', usernameVariable: 'DEV_USERNAME', passwordVariable: 'DEV_PASSWORD')]) {
                    sh 'apictl login $DEV_ENV -u $DEV_USERNAME -p $DEV_PASSWORD -k'                        
                }
                echo 'Deploying to $DEV_ENV'
                sh 'apictl import-api -f $CURR_DIR$API_DIR -e $DEV_ENV -k --preserve-provider --update --verbose'
            }
        }
        stage('Run Tests') {
            steps {
                echo 'Running tests in $DEV_ENV'
                sh 'newman run $CURR_DIR/$TEST_SCRIPT_FILE --insecure' 
            }
        }
        
        stage('Deploy to Production') {
            environment{
                RETRY = '60'
            }
            steps {
                sh 'echo "Logging into $PROD_ENV"'
                withCredentials([usernamePassword(credentialsId: 'prod_apim_admin', usernameVariable: 'PROD_USERNAME', passwordVariable: 'PROD_PASSWORD')]) {
                    sh 'apictl login $PROD_ENV -u $PROD_USERNAME -p $PROD_PASSWORD -k'                        
                }
                echo 'Deploying to Production'
                sh 'apictl import-api -f $CURR_DIR$API_DIR -e $PROD_ENV -k --preserve-provider --update --verbose'
            }
        }
    
    }
    
    post {
        cleanup {
            deleteDir()
            dir("${workspace}@tmp") {
                deleteDir()
            }
            dir("${workspace}@script") {
                deleteDir()
            }
        }
    }
   
}
