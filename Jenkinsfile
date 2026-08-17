pipeline {
    agent { label 'docker' }

    parameters {
        string(
            name: 'HOST',
            defaultValue: '',
            description: 'Hostname or IP address of the Pi-hole host'
        )
        credentials(
            name: 'SSH_CREDENTIALS',
            description: 'SSH credentials (username + password) for the Pi-hole host',
            credentialType: 'com.cloudbees.plugins.credentials.impl.UsernamePasswordCredentialsImpl',
            required: true
        )
    }

    stages {
        stage('Update Pi-hole Gravity') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: params.SSH_CREDENTIALS,
                        usernameVariable: 'SSH_USER',
                        passwordVariable: 'SSH_PASS'
                    )
                ]) {
                    script {
                        currentBuild.displayName = "#${env.BUILD_NUMBER} - ${params.HOST}"
                        currentBuild.description = "Pi-hole gravity update on ${params.HOST}"

                        def sshOpts = '-o StrictHostKeyChecking=no -o BatchMode=no'

                        // Check whether docker requires sudo on the remote host
                        def needsSudo = sh(
                            script: """
                                SSHPASS="\${SSH_PASS}" sshpass -e ssh ${sshOpts} "\${SSH_USER}@${params.HOST}" \
                                    'docker info > /dev/null 2>&1 && echo false || echo true'
                            """,
                            returnStdout: true
                        ).trim()

                        // sudo -S reads password from stdin; sshpass -e uses env var, leaving stdin free for this
                        def sshPassPipe = (needsSudo == 'true') ? 'echo "${SSH_PASS}" | ' : ''
                        def dockerCmd = (needsSudo == 'true') ? 'sudo -S docker' : 'docker'
                        echo "Using docker command: ${dockerCmd}"

                        // Collect IDs of all running pihole/pihole containers
                        def containerList = sh(
                            script: """
                                ${sshPassPipe}SSHPASS="\${SSH_PASS}" sshpass -e ssh ${sshOpts} "\${SSH_USER}@${params.HOST}" \
                                    '${dockerCmd} ps --format "{{.ID}} {{.Image}}" | grep "pihole/pihole" | cut -d" " -f1'
                            """,
                            returnStdout: true
                        ).trim()

                        if (!containerList) {
                            error("No running pihole/pihole container found on host: ${params.HOST}")
                        }

                        def containerIds = containerList.split('\n')
                        echo "Found ${containerIds.size()} Pi-hole container(s): ${containerIds.join(', ')}"

                        // Run gravity update on every container
                        containerIds.each { containerId ->
                            echo "Updating gravity on container: ${containerId}"

                            def output = sh(
                                script: """
                                    ${sshPassPipe}SSHPASS="\${SSH_PASS}" sshpass -e ssh ${sshOpts} "\${SSH_USER}@${params.HOST}" \
                                        '${dockerCmd} exec ${containerId} pihole -g'
                                """,
                                returnStdout: true
                            )

                            echo output

                            def gravityLine = output.readLines().find { it.contains('Number of gravity domains') }
                            
                            if (gravityLine) {
                                currentBuild.description += "\n${gravityLine.trim()}"
                            }
                        }
                    }
                }
            }
        }
    }
}
