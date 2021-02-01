def dockerhub = 'inyilis/kfrontend'
def image_name = "${dockerhub}:${BRANCH_NAME}"
def builder 

pipeline {

    agent any

    parameters {
        string(name: 'DOCKERHUB', defaultValue: "${image_name}", description: 'by Inyilis Punya')
        booleanParam(name: 'RUNTEST', defaultValue: 'false', description: 'Testing image')
        choice(name: 'DEPLOY', choices: ['yes', 'no'], description: 'Build pakai param')
    }

    stages {
        stage("Install dependencies")  {
            steps {
                nodejs("node14") {
                    sh 'npm install'
                }
            }
        }
        stage("Build docker image")  {
            steps {
                script {
                    builder = docker.build(image_name, "--no-cache .") 
                }
            }
        }
        stage("Testing image")  {
            when {
                expression {
                    params.RUNTEST
                }
            }
            steps {
                script {
                    builder.inside {
                        sh 'echo passed'
                    }
                }
            }
        }
        stage("Push image")  {
            steps {
                script {
                    builder.push()
                }
            }
        }
        stage("Deploy")  {
            when {
                expression {
                    params.DEPLOY == 'yes'
                }
            }
            steps {
                script {
                    if(BRANCH_NAME == 'master'){
                        sshPublisher (
                            publishers: [
                                sshPublisherDesc(
                                    configName: 'k8s',
                                    verbose: true,
                                    transfers: [
                                        sshTransfer(
                                            execCommand: "cd /home/k8s/app; echo ' ' | sudo -S kubectl apply -f dev.yml",
                                            execTimeout: 1200000
                                        )
                                    ] 
                                )
                            ]
                        )
                    }
                    if(BRANCH_NAME == 'main'){
                        sshPublisher (
                            publishers: [
                                sshPublisherDesc(
                                    configName: 'k8s',
                                    verbose: true,
                                    transfers: [
                                        sshTransfer(
                                            execCommand: "cd /home/k8s/app; echo ' ' | sudo -S kubectl apply -f prod.yml",
                                            execTimeout: 1200000
                                        )
                                    ] 
                                )
                            ]
                        )
                    }
                }
            }
        }
    }
}