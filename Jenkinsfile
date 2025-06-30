node('slave-1') {

    def mavenHome, mavenCMD, docker, dockerCMD, tagName

    stage('prepare environment') {
        mavenHome = tool name: 'maven', type: 'maven'
        mavenCMD  = "${mavenHome}/bin/mvn"
        docker     = tool name: 'docker', type: 'org.jenkinsci.plugins.docker.commons.tools.DockerTool'
        dockerCMD  = "${docker}/bin/docker"
        tagName    = "insure-me"
        echo "Using tag: ${tagName}"
    }

    stage('checkout') {
        try {
            git 'https://github.com/Rohitpotdar01/star-agile-insurance-project.git'
        } catch (e) {
            emailext body: "Git checkout failed for ${JOB_NAME} #${BUILD_NUMBER}\n${BUILD_URL}",
                     subject: "${JOB_NAME} Build #${BUILD_NUMBER} — Git Failure",
                     to: 'Rohitpotdat55@gmail.com'
            error("Stopping: Git checkout failed")
        }
    }

    stage('build') {
        sh "${mavenCMD} clean package"
    }

    stage('report') {
        publishHTML([
            reportDir: 'target/surefire-reports',
            reportFiles: 'index.html',
            reportName: 'HTML Report'
        ])
    }

    stage('docker build & push') {
        sh "${dockerCMD} rmi -f rohitpotdar/insure-me:${tagName} || true"
        sh "${dockerCMD} build -t rohitpotdar/insure-me:${tagName} ."
        withCredentials([usernamePassword(credentialsId: 'dock-password',
                                         usernameVariable: 'DOCKER_USER',
                                         passwordVariable: 'DOCKER_PASS')]) {
            sh """
               echo "$DOCKER_PASS" | ${dockerCMD} login -u "$DOCKER_USER" --password-stdin
               ${dockerCMD} push rohitpotdar/insure-me:${tagName}
            """
        }
    }

    stage('deploy via Ansible') {
        // wrap in catchError so that plugin exceptions become controlled failures
        catchError(buildResult: 'FAILURE', stageResult: 'FAILURE') {
            ansiblePlaybook(
                playbook:   'ansible-playbook.yml',
                inventory:  '/etc/ansible/hosts',
                become:     true,
                becomeUser: 'root',
                disableHostKeyChecking: true,
                installation: 'ansible',
                credentialsId: 'ansible-key'
            )
        }
    }

    // only mark SUCCESS explicitly if you want Jenkins not to consider aborted builds
    currentBuild.result = 'SUCCESS'
}
