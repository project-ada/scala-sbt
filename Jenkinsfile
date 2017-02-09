node {
  def image_name = 'adaengineering/scala-sbt'
  stage 'Checkout'
  checkout scm
  sh 'git --no-pager log -1 --format=%an > committer.txt'
  def changes_by = readFile('committer.txt').trim()
  try {
    stage 'Build'
    def app = docker.build("${image_name}")

    stage 'Push'
    app.push("${env.BRANCH_NAME}-${env.BUILD_NUMBER}")
    app.push("latest")
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'd72bb5b4-e183-4a45-b64b-2502290a24a6', passwordVariable: 'SLACK_TOKEN', usernameVariable: 'SLACK_DOMAIN']]) {
      slackSend channel: '#ci-cd', color: 'good', message: "Pushed ${image_name}:${env.BRANCH_NAME}-${env.BUILD_NUMBER} with changes by ${changes_by}", teamDomain: "${env.SLACK_DOMAIN}", token: "${env.SLACK_TOKEN}"
    }

    stage 'Clean'
    if ("${env.BRANCH_NAME}" != "master") {
      deleteDir()
    }

  } catch (err) {
    withCredentials([[$class: 'UsernamePasswordMultiBinding', credentialsId: 'd72bb5b4-e183-4a45-b64b-2502290a24a6', passwordVariable: 'SLACK_TOKEN', usernameVariable: 'SLACK_DOMAIN']]) {
      slackSend channel: '#ci-cd', color: 'danger', message: "Failure: ${err}\n(changes by ${changes_by})\n${env.BUILD_URL}", teamDomain: "${env.SLACK_DOMAIN}", token: "${env.SLACK_TOKEN}"
    }
  }
}
