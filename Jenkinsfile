// Jenkins pipeline that exercises the ReARM integration plugin against
// psclaude.rearmhq.com. Sister to the (currently-disabled) GitHub Actions
// workflow in .github/workflows/build.yml — only one of the two should be
// firing at a time.

pipeline {
    agent any

    environment {
        REARM_API = credentials('REARM_API')
        // DISABLED 2026-06-07: GitHub Actions (.github/workflows/build.yml) was
        // switched back on as the active release lane. Per the header note,
        // only one of GHA / Jenkins should fire at a time, so this pipeline is
        // gated OFF to avoid both racing to create the same ReARM release
        // version ("version already belongs to another non-pending release").
        // Flip back to 'true' to re-enable — nothing below is removed.
        JENKINS_PIPELINE_ENABLED = 'false'
    }

    stages {
        stage('Capture commit metadata') {
            when { environment name: 'JENKINS_PIPELINE_ENABLED', value: 'true' }
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
            when { environment name: 'JENKINS_PIPELINE_ENABLED', value: 'true' }
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
                        echo "ReARM minted version: ${env.VERSION} (PENDING)"
                        echo "Docker-safe version: ${env.DOCKER_VERSION}"
                        // withRearm created the release in PENDING; this call
                        // attaches metadata and flips lifecycle to ASSEMBLED.
                        addRearmRelease()
                    }
                }
            }
        }
    }
}
