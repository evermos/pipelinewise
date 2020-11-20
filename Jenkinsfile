podTemplate(containers: [
    containerTemplate(name: 'docker', image: 'docker:19.03.6', command: 'cat', ttyEnabled: true)
  ],
  volumes: [
    hostPathVolume(mountPath: '/var/run/docker.sock', hostPath: '/var/run/docker.sock')
  ]
  ) {
    node(POD_LABEL) {
        def appFullName
        def appName = "pipelinewise"
        def revision
        def message
        stage('Checkout') {
            def scmVars = checkout([
                $class: 'GitSCM',
                branches: scm.branches,
                extensions: scm.extensions + [
                    [
                        $class: 'AuthorInChangelog'
                    ],
                    [
                        $class: 'ChangelogToBranch',
                        options: [
                            compareRemote: 'origin',
                            compareTarget: 'master'
                        ]
                    ]
                ],
                userRemoteConfigs: scm.userRemoteConfigs
                ])
            appFullName = "${appName}:${scmVars.GIT_COMMIT}"
            revision = "${scmVars.GIT_COMMIT}"
            message = sh(returnStdout: true, script: "git log --oneline -1 ${revision}")
        }


        stage('Build Docker Image') {
            container('docker') {
                docker.withRegistry('https://107126629234.dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:49feb1c9-1719-4520-aa17-67695b347b0e') {
                    script {
                        sh """docker build --network=host -f Dockerfile -t 107126629234.dkr.ecr.ap-southeast-1.amazonaws.com/${appFullName} ."""
                    }
                }
            }
        }

        if (env.BRANCH_NAME == 'master') {
            stage('Push Docker Image') {
                container('docker') {
                    docker.withRegistry('https://107126629234.dkr.ecr.ap-southeast-1.amazonaws.com', 'ecr:ap-southeast-1:49feb1c9-1719-4520-aa17-67695b347b0e') {
                        script {
                            sh """docker push 107126629234.dkr.ecr.ap-southeast-1.amazonaws.com/${appFullName}"""
                        }
                    }
                }
            }
        }

        stage('Notification') {
            discordSend description: "${message}", footer: "${appFullName}", result: currentBuild.currentResult, title: "$JOB_NAME", webhookURL: "$DISCORD_WEBHOOK"
        }
    }
}