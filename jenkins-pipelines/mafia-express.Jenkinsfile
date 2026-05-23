// Simulated build pipeline for the mafia-express component.
// Generates a deterministic fake SHA-256 and submits a ReARM release with
// a CONTAINER outbound deliverable. No real Docker build/push.

pipeline {
    agent any
    environment { REARM_API = credentials('REARM_API') }

    stages {
        stage('Capture commit + simulate build') {
            steps {
                script {
                    env.COMMIT_TIME    = sh(returnStdout: true,
                        script: "git log -1 --date=iso-strict --pretty='%ad'").trim()
                    env.COMMIT_MESSAGE = sh(returnStdout: true,
                        script: "git log -1 --pretty=%s").trim()
                    env.COMMIT_AUTHOR  = sh(returnStdout: true,
                        script: "git log -1 --pretty=%an").trim()
                    env.COMMIT_EMAIL   = sh(returnStdout: true,
                        script: "git log -1 --pretty=%ae").trim()
                    env.SIM_COMMIT = sh(returnStdout: true,
                        script: "printf '%s' '${env.BUILD_NUMBER}-mafia-express-${env.GIT_COMMIT}' | sha1sum | cut -d' ' -f1").trim()
                    env.SIM_DIGEST = "sha256:" + sh(returnStdout: true,
                        script: "printf '%s' '${env.BUILD_NUMBER}-mafia-express-${env.GIT_COMMIT}' | sha256sum | cut -d' ' -f1").trim()
                    echo "Simulated mafia-express commit: ${env.SIM_COMMIT}"
                    echo "Simulated mafia-express image digest: ${env.SIM_DIGEST}"
                }
            }
        }

        stage('ReARM version + release') {
            steps {
                withRearm(
                    uri:                                       'https://psclaude.rearmhq.com',
                    vcsUri:                                    'https://github.com/relizaio/card-shuffle-claude',
                    repoPath:                                  'mafia-express',
                    commitHash:                                env.SIM_COMMIT,
                    createComponentIfMissing:                  'true',
                    createComponentName:                       'card-shuffle-claude/mafia-express (Jenkins test)',
                    createComponentVersionSchema:              'semver',
                    createComponentFeatureBranchVersionSchema: 'Branch.Micro'
                ) {
                    script {
                        echo "ReARM minted version: ${env.VERSION} (PENDING)"
                        addRearmRelease(
                            deliverableId:     "registry.psclaude.local/jenkins-test/mafia-express:${env.DOCKER_VERSION}",
                            deliverableType:   'CONTAINER',
                            deliverableDigest: env.SIM_DIGEST
                        )
                    }
                }
            }
        }
    }
}
