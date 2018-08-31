def abort_previous_running_builds() {
  def hi = Hudson.instance
  def projectname = env.JOB_NAME.split('/')[0]
  def repository = env.JOB_NAME.split('/')[1]
  hi.getItem(projectname).getItem(repository).getItem("${env.BRANCH_NAME}").getBuilds().each{ build ->
    def exec = build.getExecutor()
    if (build.number < currentBuild.number && exec != null) {
      exec.interrupt(
        Result.ABORTED,
        new CauseOfInterruption.UserInterruption(
          "Aborted by #${currentBuild.number}"
        )
      )
      println("Aborted previous running build #${build.number}")
    }
  }
}
  
def run_build(image_name, changes_by){
  def app
  try {
    stage('Build') {
      container('docker') {
        app = docker.build("${image_name}:${env.BRANCH_NAME}")
      }
    }
    stage('Push'){
      container('docker') {
        docker.withRegistry('https://registry.hub.docker.com', 'dockerhub') {
          app.push("${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
          app.push("${env.BRANCH_NAME}-latest")
          withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'd72bb5b4-e183-4a45-b64b-2502290a24a6', passwordVariable: 'SLACK_TOKEN', usernameVariable: 'SLACK_DOMAIN']]) {
            slackSend channel: '#ci-cd', color: 'good', message: "Pushed ${image_name}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} with changes by ${changes_by}", teamDomain: "${env.SLACK_DOMAIN}", token: "${env.SLACK_TOKEN}"
          }
        }
      }
    }
  }catch (err) {
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'd72bb5b4-e183-4a45-b64b-2502290a24a6', passwordVariable: 'SLACK_TOKEN', usernameVariable: 'SLACK_DOMAIN']]) {
      slackSend channel: '#ci-cd', color: 'danger', message: "Failure: ${err}\n(changes by ${changes_by})\n${env.BUILD_URL}", teamDomain: "${env.SLACK_DOMAIN}", token: "${env.SLACK_TOKEN}"
    }
  }
}

node('master') {
  stage('Startup'){
    sh "python /usr/local/bin/upgrade_docker.py"
    abort_previous_running_builds()
  }
}
podTemplate(label: 'jenkins-slave-docker',
  containers: [containerTemplate(name: 'docker', image: 'adaengineering/jenkins_slave:slave-latest',
                                 resourceRequestMemory: '512Mi', ttyEnabled: true, command: 'cat')],
  volumes: [hostPathVolume(hostPath: '/var/run/docker.sock', mountPath: '/var/run/docker.sock')]
  ) {
  def image_name = 'adaengineering/scala-sbt'
  node('jenkins-slave-docker') {
    def changes_by
    stage('Checkout'){
      checkout scm
      sh 'git --no-pager log -1 --format=%an > committer.txt'
      changes_by = readFile('committer.txt').trim()
    }
    run_build(image_name, changes_by)
  }
}
