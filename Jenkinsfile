pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'ghcr.io/seerr-team/seerr:latest'
        CONTAINER_NAME = 'seerr'
        PUID = '1000'
        PGID = '1000'
        TZ = 'America/Monterrey'
        WEBUI_PORTS = '5055/tcp,5055/udp'
        CONFIG_PATH = '/home/docker/seerr/config'
    }

    parameters {
        choice(name: 'ACTION', choices: ['Deploy', 'Update', 'Stop and Run'], description: 'Select action')
    }
    
    stages {
        stage('Pull Latest Image') {
            when {
                expression { ACTION == 'Update' }
            }
            steps {
                script {
                    sh "docker pull ${DOCKER_IMAGE}"
                }
            }
        }

        stage('Check Existing Container') {
            steps {
                script {
                    env.CONTAINER_EXISTS = sh(
                        script: "docker ps -aq -f name=${CONTAINER_NAME}",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('Perform Selected Action') {
            steps {
                script {
                    if (ACTION == 'Update' && CONTAINER_EXISTS) {
                        echo "Updating container ${CONTAINER_NAME}"
                        
                        sh """
                        docker stop ${CONTAINER_NAME}
                        docker rm ${CONTAINER_NAME}
                        docker run -d \
                            --restart always \
                            --name ${CONTAINER_NAME} \
                            -p 5055:5055 \
                            -e PUID=${PUID} \
                            -e PGID=${PGID} \
                            -e TZ=${TZ} \
                            -v ${CONFIG_PATH}:/config \
                            -v /mnt/Media:/Media \
                            -v /mnt/Media/Downloads:/downloads \
                            -v /mnt/Media/TVShows:/TVShows \
                            -v /mnt/Media/MyShows:/MyShows \
                            -v /mnt/Media/Movies:/Movies \
                            -v /mnt/Media/MyMovies:/MyMovies \
                            ${DOCKER_IMAGE}
                        """
                    } else if (ACTION == 'Stop and Run' && CONTAINER_EXISTS) {
                        echo "Restarting container ${CONTAINER_NAME} without updating image"
                        
                        sh """
                        docker stop ${CONTAINER_NAME}
                        docker start ${CONTAINER_NAME}
                        """
                    } else {
                        echo "Deploying new container ${CONTAINER_NAME}"
                        
                        sh """                      
                        docker run -d \
                            --restart always \
                            --name ${CONTAINER_NAME} \
                            -p 5055:5055 \
                            -e PUID=${PUID} \
                            -e PGID=${PGID} \
                            -e TZ=${TZ} \
                            -v ${CONFIG_PATH}:/config \
                            -v /mnt/Media:/Media \
                            -v /mnt/Media/Downloads:/downloads \
                            -v /mnt/Media/TVShows:/TVShows \
                            -v /mnt/Media/MyShows:/MyShows \
                            -v /mnt/Media/Movies:/Movies \
                            -v /mnt/Media/MyMovies:/MyMovies \
                            ${DOCKER_IMAGE}
                        """
                    }
                }
            }
        }
    }
}
