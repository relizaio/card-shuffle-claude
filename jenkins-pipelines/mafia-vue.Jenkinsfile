// Simulated build pipeline for the mafia-vue component.
// Does NOT actually build/push the Docker image — generates a deterministic
// fake SHA-256 keyed on (BUILD_NUMBER, GIT_COMMIT, component) and submits
// a ReARM release with an outbound deliverable (CONTAINER) carrying that
// digest. Each rebuild bumps the version and produces a new release/
// deliverable row on psclaude.

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
                    env.SIM_DIGEST = "sha256:" + sh(returnStdout: true,
                        script: "printf '%s' '${env.BUILD_NUMBER}-mafia-vue-${env.GIT_COMMIT}' | sha256sum | cut -d' ' -f1").trim()
                    echo "Simulated mafia-vue image digest: ${env.SIM_DIGEST}"
                }
            }
        }

        stage('ReARM version + release') {
            steps {
                withRearm(
                    uri:                                       'https://psclaude.rearmhq.com',
                    vcsUri:                                    'https://github.com/relizaio/card-shuffle-claude',
                    repoPath:                                  'mafia-vue',
                    createComponentIfMissing:                  'true',
                    createComponentName:                       'card-shuffle-claude/mafia-vue (Jenkins test)',
                    createComponentVersionSchema:              'semver',
                    createComponentFeatureBranchVersionSchema: 'Branch.Micro'
                ) {
                    script {
                        echo "ReARM minted version: ${env.VERSION} (PENDING)"
                        addRearmRelease(
                            deliverableId:     "registry.psclaude.local/jenkins-test/mafia-vue:${env.DOCKER_VERSION}",
                            deliverableType:   'CONTAINER',
                            deliverableDigest: env.SIM_DIGEST
                        )
                    }
                }
            }
        }
    }
}
