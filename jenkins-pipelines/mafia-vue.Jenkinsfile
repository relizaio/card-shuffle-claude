// Simulated build pipeline for the mafia-vue component.
// Generates a deterministic fake digest, mints a PENDING release via
// withRearm, and finalizes via addRearmRelease — attaching:
//   * an SCE-scoped BOM (sceArts)
//   * a release-scoped BOM (releaseArts)
//   * a deliverable-scoped BOM (odelArts) which itself has a child
//     SIGNATURE artifact (artifact-of-artifact) — fake content, just
//     verifying the nested-upload path works.

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
                        script: "printf '%s' '${env.BUILD_NUMBER}-mafia-vue-${env.GIT_COMMIT}' | sha1sum | cut -d' ' -f1").trim()
                    env.SIM_DIGEST = "sha256:" + sh(returnStdout: true,
                        script: "printf '%s' '${env.BUILD_NUMBER}-mafia-vue-${env.GIT_COMMIT}' | sha256sum | cut -d' ' -f1").trim()
                    echo "Simulated mafia-vue commit: ${env.SIM_COMMIT}"
                    echo "Simulated mafia-vue image digest: ${env.SIM_DIGEST}"

                    // Three CycloneDX 1.6 BOMs that differ only by serialNumber.
                    // Each is small but valid enough for ReARM to accept as a
                    // CYCLONEDX BOM upload. UUIDs derived from BUILD_NUMBER so
                    // re-runs always produce fresh serials.
                    def bomDir = "${env.WORKSPACE}/artifacts"
                    sh "mkdir -p '${bomDir}'"
                    for (scope in ['sce', 'release', 'odel']) {
                        def uuid = sh(returnStdout: true,
                            script: "printf '%s' '${env.BUILD_NUMBER}-${scope}-mafia-vue' | sha256sum | head -c 32").trim()
                        def bom = """{
  "bomFormat": "CycloneDX",
  "specVersion": "1.6",
  "serialNumber": "urn:uuid:${uuid.substring(0,8)}-${uuid.substring(8,12)}-${uuid.substring(12,16)}-${uuid.substring(16,20)}-${uuid.substring(20,32)}",
  "version": 1,
  "metadata": {
    "timestamp": "${env.COMMIT_TIME}",
    "component": {
      "type": "application",
      "name": "mafia-vue",
      "version": "${env.DOCKER_VERSION ?: '0.0.0'}",
      "purl": "pkg:generic/mafia-vue@${env.DOCKER_VERSION ?: '0.0.0'}?scope=${scope}"
    }
  },
  "components": [
    {"type": "library", "name": "vue", "version": "3.x", "purl": "pkg:npm/vue@3.x"}
  ]
}"""
                        writeFile file: "${bomDir}/mafia-vue.${scope}.cdx.json", text: bom
                    }
                    writeFile file: "${bomDir}/signature.txt",
                              text: "FAKE-SIG build=${env.BUILD_NUMBER} digest=${env.SIM_DIGEST}\nnot-a-real-signature\n"
                    sh "ls -la '${bomDir}'"
                }
            }
        }

        stage('ReARM version + release + artifacts') {
            steps {
                withRearm(
                    uri:                                       'https://psclaude.rearmhq.com',
                    vcsUri:                                    'https://github.com/relizaio/card-shuffle-claude',
                    repoPath:                                  'mafia-vue',
                    commitHash:                                env.SIM_COMMIT,
                    createComponentIfMissing:                  'true',
                    createComponentName:                       'card-shuffle-claude/mafia-vue (Jenkins test)',
                    createComponentVersionSchema:              'semver',
                    createComponentFeatureBranchVersionSchema: 'Branch.Micro'
                ) {
                    script {
                        echo "ReARM minted version: ${env.VERSION} (PENDING)"
                        def base = "${env.WORKSPACE}/artifacts"

                        def sceArts = groovy.json.JsonOutput.toJson([
                            [
                                displayIdentifier: "mafia-vue-${env.DOCKER_VERSION}.sce.cdx.json",
                                type:              'BOM',
                                bomFormat:         'CYCLONEDX',
                                filePath:          "${base}/mafia-vue.sce.cdx.json"
                            ]
                        ])
                        def releaseArts = groovy.json.JsonOutput.toJson([
                            [
                                displayIdentifier: "mafia-vue-${env.DOCKER_VERSION}.release.cdx.json",
                                type:              'BOM',
                                bomFormat:         'CYCLONEDX',
                                filePath:          "${base}/mafia-vue.release.cdx.json"
                            ]
                        ])
                        // The odelArts BOM carries a child SIGNATURE artifact —
                        // tests the artifact-of-artifact upload path. Both
                        // file paths get resolved into Apollo multipart parts.
                        def odelArts = groovy.json.JsonOutput.toJson([
                            [
                                displayIdentifier: "mafia-vue-${env.DOCKER_VERSION}.odel.cdx.json",
                                type:              'BOM',
                                bomFormat:         'CYCLONEDX',
                                filePath:          "${base}/mafia-vue.odel.cdx.json",
                                artifacts: [
                                    [
                                        displayIdentifier: "mafia-vue-${env.DOCKER_VERSION}.odel.cdx.json.sig",
                                        type:              'SIGNATURE',
                                        filePath:          "${base}/signature.txt"
                                    ]
                                ]
                            ]
                        ])

                        addRearmRelease(
                            deliverableId:     "registry.psclaude.local/jenkins-test/mafia-vue:${env.DOCKER_VERSION}",
                            deliverableType:   'CONTAINER',
                            deliverableDigest: env.SIM_DIGEST,
                            sceArtsJson:       sceArts,
                            releaseArtsJson:   releaseArts,
                            odelArtsJson:      odelArts
                        )
                    }
                }
            }
        }
    }
}
