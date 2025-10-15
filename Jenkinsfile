pipeline {
  agent any
  options { disableConcurrentBuilds(); timestamps() }

  parameters {
    string(name: 'IMAGE_NAME', defaultValue: 'murilothebr/tcc-cicd-comparativo', description: 'Docker image repo (ex.: username/repo)', trim: true)
  }

  environment {
    IMAGE_NAME = "${params.IMAGE_NAME}"
    SOLUTION_PATH = "src/Api.sln"
    DOTNET_SKIP_FIRST_TIME_EXPERIENCE = '1'
    DOTNET_CLI_TELEMETRY_OPTOUT = '1'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Restore/Build/Test (.NET 8)') {
      steps {
        sh '''
          docker run --rm \
            --volumes-from jenkins \
            -e SOLUTION_PATH="$SOLUTION_PATH" \
            -e DOTNET_SKIP_FIRST_TIME_EXPERIENCE="$DOTNET_SKIP_FIRST_TIME_EXPERIENCE" \
            -e DOTNET_CLI_TELEMETRY_OPTOUT="$DOTNET_CLI_TELEMETRY_OPTOUT" \
            -w "$WORKSPACE" \
            mcr.microsoft.com/dotnet/sdk:8.0 bash -lc '
              set -e
              TARGET="$SOLUTION_PATH"
              if [ -z "$TARGET" ] || [ ! -f "$TARGET" ]; then
                TARGET="$(find . -maxdepth 6 -name "*.sln" | head -n1 || true)"
                [ -n "$TARGET" ] || TARGET="$(find . -maxdepth 6 -path "./src/*" -name "*.csproj" | head -n1 || true)"
                [ -n "$TARGET" ] || { echo "No .sln or .csproj found."; exit 1; }
              fi
              dotnet --info
              dotnet restore "$TARGET"
              dotnet build "$TARGET" --configuration Release --no-restore
              if echo "$TARGET" | grep -qi "\\.sln$"; then
                dotnet test "$TARGET" --configuration Release --no-build --logger "trx;LogFileName=test-results.trx"
              else
                TESTS=$(find tests -maxdepth 6 -name "*.csproj" || true)
                [ -z "$TESTS" ] || for t in $TESTS; do dotnet test "$t" --configuration Release --no-build --logger "trx;LogFileName=test-results.trx"; done
              fi
            '
        '''
      }
      post {
        always {
          archiveArtifacts artifacts: '**/*.trx', fingerprint: true, allowEmptyArchive: true
        }
      }
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
