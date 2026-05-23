// Jenkins pipeline that exercises the ReARM integration plugin against
// psclaude.rearmhq.com. Sister to the (currently-disabled) GitHub Actions
// workflow in .github/workflows/build.yml — only one of the two should be
// firing at a time.

pipeline {
    agent any

    environment {
        REARM_API = credentials('REARM_API')
    }

    options {
        timestamps()
        ansiColor('xterm')
    }

    stages {
        stage('Capture commit metadata') {
            steps {
                script {
                    env.COMMIT_TIME = sh(
                        script: "git log -1 --date=iso-strict --pretty='%ad'",
                        returnStdout: true
                    ).trim()
                    env.COMMIT_MESSAGE = sh(
                        script: "git log -1 --pretty=%s",
                        returnStdout: true
                    ).trim()
                }
            }
        }

        stage('ReARM version + release') {
            steps {
                withRearm(
                    uri: 'https://psclaude.rearmhq.com',
                    vcsUri: 'https://github.com/relizaio/card-shuffle-claude',
                    repoPath: 'jenkins-test',
                    createComponentIfMissing: 'true',
                    createComponentName: 'card-shuffle-claude (Jenkins test)',
                    createComponentVersionSchema: 'semver',
                    createComponentFeatureBranchVersionSchema: 'Branch.Micro',
                    jenkinsVersionMeta: 'true'
                ) {
                    script {
                        echo "ReARM minted version: ${env.VERSION}"
                        echo "Docker-safe version: ${env.DOCKER_VERSION}"
                        env.STATUS = 'complete'
                        addRearmRelease()
                    }
                }
            }
        }
    }
}
