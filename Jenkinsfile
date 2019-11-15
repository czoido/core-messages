
def get_stages(id, docker_image, artifactory_name, artifactory_repo, profile) {
    return {
        node {
            docker.image(docker_image).inside("--network host") {
                //def server = Artifactory.server artifactory_name
                //def client = Artifactory.newConanClient()
                def remoteName = "artifactory-local"
                def lockfile = "${id}.lock"
                //def buildInfo = Artifactory.newBuildInfo()
                def buildInfoFilename = "${id}.json"

                try {
                    sh "conan config install -sf conan/config https://github.com/sword-and-sorcery/sword-and-sorcery.git"
                    sh "conan config install -sf hooks -tf hooks https://github.com/conan-io/hooks.git"
                    //sh "conan remote remove artifactory-local"
                    sh "uname -a"
                    //sh "conan remote add artifactory-local http://host.docker.internal:8090/artifactory"
                    sh "conan user -p password -r artifactory-local admin"

                    stage("${id}") {
                        echo 'Running in ${docker_image}'
                    }

                    stage("Get project") {
                        checkout scm
                    }

                    stage("Start build info") {
                        String start_build_info = "conan_build_info --v2 start \"${env.JOB_NAME}\" ${env.BUILD_NUMBER}"
                        sh start_build_info
                    }

                    stage("Get dependencies and create app") {
                        String arguments = "--profile ${profile} --lockfile=${lockfile}"
                        sh "conan graph lock . ${arguments}"
                        sh "conan create . sword/sorcery ${arguments} --build missing"
                        sh "conan search '*''"
                        sh "conan search core-messages/0.0@sword/sorcery"
                        sh "cat ${lockfile}"

                        String uploadCommand = "conan upload '*' --all -r ${remoteName} --confirm --force"
                        sh uploadCommand

                        sh "conan config home"
                        sh "search '*'"
                        String create_build_info = "conan_build_info --v2 create --lockfile ${lockfile} --user admin --password password ${buildInfoFilename}"
                        sh create_build_info

                        String publish_build_info = "conan_build_info --v2 publish --url http://host.docker.internal:8090/artifactory --user admin --password password ${buildInfoFilename}"
                        sh publish_build_info

                    }

                    // stage("Upload packages") {
                    // }

                    // stage("Create build info") {
                    // }

                    // stage("Publish build info") {
                    // }

                    stage("Compute build info") {
                        // def buildInfo = Artifactory.newBuildInfo()
                        // String artifactory_credentials = "http://artifactory:8081/artifactory,admin,password"
                        // def buildInfoFilename = "${id}.json"

                        // // Install helper script (WIP)
                        // git url: 'https://gist.github.com/a39acad525fd3e7e5315b2fa0bc70b6f.git'
                        // sh 'pip install rtpy'

                        // String python_command = "python lockfile_buildinfo.py --remotes=${artifactory_credentials}"
                        // python_command += " --build-number=${buildInfo.getNumber()} --build-name=\"${buildInfo.getName()}\""
                        // python_command += " --multi-module"
                        // python_command += " --output-file=${buildInfoFilename} ${lockfile}"
                        // sh python_command

                        // echo "Stash '${id}' -> '${buildInfoFilename}'"
                        // stash name: id, includes: "${buildInfoFilename}"
                    }
                }
                finally {
                    deleteDir()
                }
            }
        }
    }
}

def artifactory_name = "Artifactory Docker"
def artifactory_repo = "conan-local"
def docker_runs = [:]  // [id] = [docker_image, profile]
docker_runs["conanio-gcc8"] = ["conanio/gcc8", "conanio-gcc8"]
//docker_runs["conanio-gcc7"] = ["conanio/gcc7", "conanio-gcc7"]

def stages = [:]
docker_runs.each { id, values ->
    stages[id] = get_stages(id, values[0], artifactory_name, artifactory_repo, values[1])
}


node {
    try {
        stage("Build + upload") {
            withEnv(["CONAN_HOOK_ERROR_LEVEL=40"]) {
                parallel stages
            }
        }

        stage("Retrieve build info") {
            docker.image("conanio/gcc8").inside("--net=docker_jenkins_artifactory") {
                //def buildInfo = Artifactory.newBuildInfo()
                // String artifactory_credentials = "http://artifactory:8081/artifactory,admin,password"
                // def buildInfoFilename = "buildinfo.json"
                
                // // Install helper script (WIP)
                // git url: 'https://gist.github.com/a39acad525fd3e7e5315b2fa0bc70b6f.git'
                // sh 'pip install rtpy'

                // // Merge Build Info from nodes
                // String merge_bi_command = "python merge_buildinfo.py --output-file ${buildInfoFilename}"
                // docker_runs.each { id, values ->
                //     unstash id
                //     merge_bi_command += " ${id}.json"
                // }
                // sh merge_bi_command

                // // Publish build info
                // String publish_command = "python publish_buildinfo.py --remote=${artifactory_credentials} ${buildInfoFilename}"
                // sh publish_command

            }
        }
    }
    finally {
        deleteDir()
    }
}
