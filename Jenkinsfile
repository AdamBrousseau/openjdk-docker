def archList = (params.ARCH_LIST) ? params.ARCH_LIST.tokenize(',') : ["x64", "aarch64", "armv7l", "ppc64le", "s390x"]
def baseLabel = (params.BASE_LABEL) ?: "dockerBuild&&linux"
def archLabelTemplate = (params.ARCH_LABEL_TEMPLATE) ?: ""
def versions = [8, 11, 15, 16]
// dockerhub is the ID of the credentials stored in Jenkins
dockerHubCredentialID = (params.DOCKERHUB_CREDENTIAL_ID) ?: "dockerhub"
dockerRegistry = (params.DOCKER_REGISTRY) ?: "https://index.docker.io/v1/"

def timeoutTime = (params.TIMEOUT_TIME) ? params.TIMEOUT_TIME.toInteger() : 30

def builds = [:]
def manifests = [:]

timestamps {
    timeout(time: timeoutTime, unit: 'HOURS') {
        for (_arch in archList) {
            def arch = _arch
            builds[arch] = {
                stage("Linux ${arch}") {
                    archLabel = (arch == "x64") ? "x86" : arch
                    node("${baseLabel}&&${archLabelTemplate}${archLabel}") {
                        try {
                            dockerBuild(null)
                        } finally {
                            cleanWs()
                        }
                    }
                }
            }
        }
        parallel builds
        for (_version in versions) {
            def version = _version
            manifests[version] {
                stage("Manifest ${version}") {
                    node("${baseLabel}&&x64") {
                        try {
                            withEnv(['DOCKER_CLI_EXPERIMENTAL = "enabled"']) {
                                dockerManifest(version)
                            }
                        } finally {
                            cleanWs()
                        }
                    }
                }
            }
        }
        parallel manifests
    }
}

def dockerBuild(version) {
    // dockerhub is the ID of the credentials stored in Jenkins
    docker.withRegistry(dockerRegistry, dockerHubCredentialID) {
        checkout scm
        if (version){
            sh label: '', script: "./build_all.sh ${version}"
        } else {
            sh label: '', script: "./build_all.sh"
        }
    }
}

def dockerManifest(version) {
    docker.withRegistry(dockerRegistry, dockerHubCredentialID) {
        checkout scm
        sh label: '', script: "./update_manifest_all.sh ${version}"
    }
}
