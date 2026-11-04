
pipeline {
    agent any

    environment {
        IMAGE_NAME = 'my-website'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        CONTAINER_NAME = 'my-website-ci'
    }

    stages {

        stage('Build image') {
            steps {
                powershell '''
                    docker build --tag "$env:IMAGE_NAME`:$env:IMAGE_TAG" .
                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed"
                    }
                '''
            }
        }

        stage('Smoke test') {
            steps {
                powershell '''
                    $ErrorActionPreference = "Stop"

                    docker rm --force $env:CONTAINER_NAME 2>$null

                    docker run --detach `
                        --publish 18080:80 `
                        --name $env:CONTAINER_NAME `
                        "$($env:IMAGE_NAME):$($env:IMAGE_TAG)"

                    if ($LASTEXITCODE -ne 0) {
                        throw "Container startup failed"
                    }

                    try {
                        $healthy = $false

                        for ($attempt = 1; $attempt -le 10; $attempt++) {
                            try {
                                $response = Invoke-WebRequest `
                                    -Uri "http://127.0.0.1:18080/" `
                                    -TimeoutSec 3

                                if ($response.StatusCode -eq 200) {
                                    $healthy = $true
                                    break
                                }
                            }
                            catch {
                                Start-Sleep -Seconds 2
                            }
                        }

                        if (-not $healthy) {
                            throw "Website smoke test failed"
                        }

                        Write-Host "Website is responding successfully!"
                    }
                    finally {
                        docker rm --force $env:CONTAINER_NAME
                    }
                '''
            }
        }
    }

    post {
        always {
            powershell '''
                docker rm --force $env:CONTAINER_NAME 2>$null
                docker image rm "$($env:IMAGE_NAME):$($env:IMAGE_TAG)" 2>$null
                exit 0
            '''
        }
    }
}
