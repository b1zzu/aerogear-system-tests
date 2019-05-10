#!groovy

def clusterName = "aerogear"
def awsRegion = "eu-west-2"
def clusterMasterUrl = "https://${clusterName}.skunkhenry.com"
def dryRun = true

pipeline {
    agent {
        node {
            label 'cirhos_rhel7'
        }
    }

    stages {
        stage('Cluster Create') {
            steps {
                echo "[INFO] Creating ${clusterName} cluster"
                build job: 'openshift-cluster-create', parameters: [
                    string(name: 'clusterName', value: clusterName),
                    string(name: 'awsRegion', value: awsRegion),
                    booleanParam(name: 'dryRun', value: dryRun),
                ]
            }
        }

        stage('Install Integreatly') {
            steps {
                echo "[INFO] Installing Integreatly in ${clusterName} cluster"
                build job: 'openshift-cluster-integreatly-install', parameters: [
                    string(name: 'clusterName', value: clusterName),
                    string(name: 'openshiftMasterUrl', value: clusterMasterUrl),
                    booleanParam(name: 'dryRun', value: dryRun),
                ]
            }
        }

        stage('Test Integreatly') {
            steps {
                echo "[INFO] Running Tests using ${clusterName} cluster"
                withCredentials([usernamePassword(
                    credentialsId: 'tower-openshift-cluster-credentials', 
                    usernameVariable: 'CLUSTER_ADMIN_USERNAME', 
                    passwordVariable: 'CLUSTER_ADMIN_PASSWORD'
                )]) {
                    sh "oc login ${clusterMasterUrl} -u ${CLUSTER_ADMIN_USERNAME} -p ${CLUSTER_ADMIN_PASSWORD}"
                }
            }
        }
    }

    post {
        always {
            echo "[INFO] Deprovision ${clusterName} cluster"
            build job: 'openshift-cluster-deprovision', parameters: [
                string(name: 'clusterName', value: clusterName),
                string(name: 'awsRegion', value: awsRegion),
                booleanParam(name: 'dryRun', value: dryRun),
            ]
        }
    }
}
