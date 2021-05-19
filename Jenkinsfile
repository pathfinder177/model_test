pipeline {
    triggers {
        //triggers every twelve hours
        cron('H */12 * * 1-5')
        //and check the changes on repo periodically
        pollSCM('H */4 * * 1-5')
    }
    
    stages {
        environment {
            MODEL_VERSION = "mode-v1.2.bin"
            REPO = "git@git.company.local:algo/test.git"
            BRANCH = "master"
            DOCKER_BUILD = "dockerRepo/python39TestModel:$TAG"
        }

        //Building a docker container with model, dataset and testing software
        //It is better to have all the dependencies inside the container
        //  and rebuild the container if there is push to repo
        stage('Build') {
            environment {
                TAG = "latest"
            }

            //Suppose we have an apart Jenkins slave building docker containers
            //  and push it to the our repo
            agent {
                label 'docker-builder'
            }

            //Try to use git plugin to checkout
            dir('test') {
                git branch: "${env.BRANCH}",
                url: "${REPO}"
            }

            steps {
                //Mount nfs-share to agent to copy dataset to the container
                //  thus we save network resources
                sh '''
                    cd ./test
                    apt-get update -y
                    apt-get install nfs-common -y
                    mkdir ./docker/dataset
                    mount 192.168.2.10:/shared/dataset ./docker/dataset
                '''

                //Push the built docker image to local docker repo 
                //  for further using it in the Test stage
                sh '''    
                    cd ./test/docker
                    echo "building $DOCKER_BUILD"
                    docker build . \
                      --build-arg MODEL_VERSION="$MODEL_VERSION" \
                      --build-arg REPO="$REPO" \
                      --build-arg BRANCH="$BRANCH" \
                      --tag "$TAG"
                    docker push "$DOCKER_BUILD"
                '''
            }
        }

        stage('Test') {
            environment {
                TAG = "latest"
                TEST_SCRIPT='docker run -e TARGET="${env.STAGE_NAME}" "${DOCKER_BUILD}"'
            }

            agent {
                label 'linux-executor'
            }

            parallel {
                stage('CPU_Linux') {
                    steps {
                        sh "${TEST_SCRIPT}"
                    }
                }
                stage('GPU_Linux') {
                    steps {
                        sh "${TEST_SCRIPT}"
                    }
                }
                stage('CPU_Windows') {
                    steps {
                        sh "${TEST_SCRIPT}"
                    }
                }
                stage('GPU_Windows') {
                    steps {
                        sh "${TEST_SCRIPT}"
                    }
                }
            }
        }
    }

     post {
        always {
            publishReport 'build/reports/*.report'
        }
    }
}