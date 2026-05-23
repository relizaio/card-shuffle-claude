// Simulated build pipeline for the mafia-helm component.
// Helm charts ship as .tgz files rather than container images, so the
// outbound deliverable is typed FILE. Still no real packaging — just a
// fake digest per (BUILD_NUMBER, GIT_COMMIT).

pipeline {
    agent any
    environment { REARM_API = credentials('REARM_API') }

    stages {
        stage('Capture commit + simulate package') {
            steps {
                script {
                    env.COMMIT_TIME    = sh(returnStdout: true,
                        script: "git log -1 --date=iso-strict --pretty='%ad'").trim()
                    env.COMMIT_MESSAGE = sh(returnStdout: true,
                        script: "git log -1 --pretty=%s").trim()
                    env.SIM_DIGEST = "sha256:" + sh(returnStdout: true,
                        script: "printf '%s' '${env.BUILD_NUMBER}-mafia-helm-${env.GIT_COMMIT}' | sha256sum | cut -d' ' -f1").trim()
                    echo "Simulated mafia-helm chart digest: ${env.SIM_DIGEST}"
                }
            }
        }

        stage('ReARM version + release') {
            steps {
                withRearm(
                    uri:                                       'https://psclaude.rearmhq.com',
                    vcsUri:                                    'https://github.com/relizaio/card-shuffle-claude',
                    repoPath:                                  'mafia-helm',
                    createComponentIfMissing:                  'true',
                    createComponentName:                       'card-shuffle-claude/mafia-helm (Jenkins test)',
                    createComponentVersionSchema:              'semver',
                    createComponentFeatureBranchVersionSchema: 'Branch.Micro'
                ) {
                    script {
                        echo "ReARM minted version: ${env.VERSION} (PENDING)"
                        addRearmRelease(
                            deliverableId:     "mafia-helm-${env.DOCKER_VERSION}.tgz",
                            deliverableType:   'FILE',
                            deliverableDigest: env.SIM_DIGEST
                        )
                    }
                }
            }
        }
    }
}
