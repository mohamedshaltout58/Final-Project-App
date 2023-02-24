

pipeline {
    agent { label 'my-final-task' }
    stages {
        stage('build') {
            steps {
                script {
                    sh   """
                        docker build -t gcr.io/gcp-labs-project/myapp:${BUILD_NUMBER} .
                        docker push gcr.io/gcp-labs-project/myapp:${BUILD_NUMBER}
                        echo ${BUILD_NUMBER} > ../proj-build-number.txt
                    """
                }
            }
        }
        stage('deploy') {
            steps {
                script {
                    def namespace = "myapp"
                    def namespaceExists = sh(returnStdout: true, script: "kubectl get ns | grep ${namespace} | wc -l").trim()
                    if (namespaceExists == "1") {
                        echo "Namespace ${namespace} already exists, skipping creation step."
                    } else {
                        sh "kubectl create namespace ${namespace}"
                    }
                    sh    """
                        export BUILD_NUMBER=\$(cat ../proj-build-number.txt)
                        mv Deployment/deploy.yaml Deployment/deploy.yaml.tmp
                        cat Deployment/deploy.yaml.tmp | envsubst > Deployment/deploy.yaml
                        rm -f Deployment/deploy.yaml.tmp
                        kubectl apply -f Deployment
                    """
                }
            }
        }
    }
}
