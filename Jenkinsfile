pipeline {
    environment {
        GH_CREDS = credentials('jenkins-x-github')
        GKE_SA = credentials('gke-sa')
        TEST_USER = credentials('test-user')
    }
    agent {
        label "jenkins-go"
    }
    stages {
        stage('CI Build') {
            when {
                branch 'PR-*'
            }
            environment {
                CLUSTER_NAME = "JXCE-$BRANCH_NAME"
                ZONE = "europe-west1-b"
                PROJECT_ID = "jenkinsx-dev"
                SERVICE_ACCOUNT_FILE = "$GKE_SA"
                TEST_PASSWORD = "$TEST_USER_PSW"
                GIT_API_TOKEN = "$GH_CREDS_PSW"
                GIT_USERNAME = "$GH_CREDS_USR"
            }
            steps {
                container('go') {
                    sh "./jx/scripts/ci-gke.sh"
                    retry(3){
                        sh "jx create jenkins user --headless --password $TEST_PASSWORD admin"
                    }

                    dir ('/home/jenkins/go/src/github.com/jenkins-x/godog-jx') {
                        git "https://github.com/jenkins-x/godog-jx"

                        sh "git config credential.helper store"

                        input "Wanna test?"
                        
                        sh "./bdd-importurl.sh"
                    }
                }
            }
        }

        stage('Build and Push Release') {
            when {
                branch 'master'
            }
            steps {
                // auto upgrade demo env
                echo 'auto upgrades not yet implemented'
            }
        }
    }
}
