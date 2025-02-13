pipeline{
    agent any
    parameters {
        string(name: 'IMAGE_NAME')
        string(name: 'NAMESPACE')
        string(name: 'Tags')
    }
    environment {
        GCLOUD_PROJECT = 'vishwas-426408'
        CLUSTER_NAME = 'jenkins-cluster'
        CLUSTER_ZONE = 'us-central1-c'
        LOCATION = 'us-central1-c	'
        CREDENTIALS_ID = 'GCP-SA'
    }
    stages{
        stage("Cloning code from GitHub"){  
            steps{
            git branch: 'main', url: 'https://github.com/Deepanshu-goswami/Java-app-CD-Pipeline.git'  
             }
            }
        stage('login to GCP') {
            steps {
                withCredentials([file(credentialsId: 'GCP-SA', variable: 'CREDENTIALS_ID')]) {
                    sh '''
                    gcloud version
                    gcloud auth activate-service-account --key-file="$CREDENTIALS_ID"
                    gcloud config set project $GCLOUD_PROJECT
                    '''
                }
            }
        }
        stage('Validate Docker Image') {
            steps {
                script {
                    def projectName = 'vishwas-426408'
                    def location = 'asia-south2'
                    def repository = 'jenkins-repo'
                    def packageName = 'java-application-new'
                    def tag = '6'

                    def cmd = "gcloud artifacts files list --project=${projectName} --location=${location} --repository=${repository} --package=${packageName} --tag=${tag}"
                    def result = sh(returnStdout: true, script: cmd)

                    if (result.trim() != "") {
                        echo "Docker image ${packageName}:${tag} exists in Artifact Registry"
                    } else {
                        error "Docker image ${packageName}:${tag} does not exist in Artifact Registry"
                    }
                }
            }
        }
        stage('Connecting to the GKE Cluster') {
                steps {
                withCredentials([file(credentialsId: 'GCP-SA', variable: 'CREDENTIALS_ID')]){
                sh '''
                gcloud auth activate-service-account --key-file="$CREDENTIALS_ID"
                gcloud config set compute/zone $CLUSTER_ZONE
                gcloud container clusters get-credentials $CLUSTER_NAME
                '''
                       }
            }
        }
        stage('Namespace Creation if Not Exists') {
            steps {
                script {
                    def exists = sh(script: "kubectl get namespace ${NAMESPACE} -o yaml", returnStatus: true) == 0
                    if (exists) {
                        echo "Namespace '${NAMESPACE}' already exists."
                    } else {
                        sh(script: "kubectl create namespace ${NAMESPACE}", returnStatus: true)
                        echo "Namespace '${NAMESPACE}' created."
                    }
                }
            }
        }
        stage("Deployment to GKE Cluster"){steps{
                sh 'kubectl apply -f deployment.yml -f ingress.yaml -n ${NAMESPACE}'
            } }
    }
    post {
        success {
            echo 'Pipeline succeeded!!'
            mail to: 'Deepanshu.goswami95@gmail.com',
                 subject: "Pipeline Succeeded: ${currentBuild.fullDisplayName}",
                 body: "Congratulations! The pipeline ${env.BUILD_URL} has successfully completed."
        }
        failure {
            echo 'Pipeline failed!'
            mail to: 'Deepanshu.goswami95@gmail.com',
                 subject: "Pipeline Failed: ${currentBuild.fullDisplayName}",
                 body: "Oops! The pipeline ${env.BUILD_URL} has failed. Check the logs for details.Try Again"
        }
    }
        }
