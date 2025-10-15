pipeline {
  agent any
  options { disableConcurrentBuilds(); timestamps() }

  parameters {
    string(name: 'IMAGE_NAME',
           defaultValue: 'murilothebr/tcc-cicd-comparativo',
           description: 'Docker image repo (ex.: username/repo)',
           trim: true)
  }

  environment {
    IMAGE_NAME = "${params.IMAGE_NAME}"
    SOLUTION_PATH = "src/Api.sln"
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE = '1'
    DOTNET_CLI_TELEMETRY_OPTOUT = '1'
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Restore/Build/Test (.NET 8)') {
        steps {
            sh '''
                docker run --rm -v "$PWD":/ws -w /ws mcr.microsoft.com/dotnet/sdk:8.0 bash -lc '
                dotnet --info &&
                dotnet restore "$SOLUTION_PATH" &&
                dotnet build "$SOLUTION_PATH" --configuration Release --no-restore &&
                dotnet test "$SOLUTION_PATH" --configuration Release --no-build --logger "trx;LogFileName=test-results.trx"'
            '''
        }
        post { always { archiveArtifacts artifacts: '**/*.trx', fingerprint: true } }
    }

    stage('Docker Build, Trivy Scan & Push') {
      when { expression { return env.IMAGE_NAME?.trim() } }
      steps {
        script {
          def gitSha = env.GIT_COMMIT ?: sh(script: 'git rev-parse HEAD', returnStdout: true).trim()

          withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'DOCKERHUB_USERNAME', passwordVariable: 'DOCKERHUB_TOKEN')]) {
            sh """
              echo "\$DOCKERHUB_TOKEN" | docker login -u "\$DOCKERHUB_USERNAME" --password-stdin
              docker build --pull -f Dockerfile -t "${IMAGE_NAME}:${gitSha}" .
              docker run --rm -v /var/run/docker.sock:/var/run/docker.sock aquasec/trivy:0.54.1 image --severity CRITICAL,HIGH --ignore-unfixed --exit-code 1 "${IMAGE_NAME}:${gitSha}"
              docker push "${IMAGE_NAME}:${gitSha}"
            """

            def onMain = (env.BRANCH_NAME == 'main') || (env.GIT_BRANCH?.endsWith('/main'))
            if (onMain) {
              sh """
                docker tag "${IMAGE_NAME}:${gitSha}" "${IMAGE_NAME}:jenkins_latest"
                docker push "${IMAGE_NAME}:jenkins_latest"
              """
            }
          }
        }
      }
    }
  }
}
