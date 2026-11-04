
pipeline {
    agent any

    environment {
        DOCKER = 'C:\\Users\\kc373\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'
        IMAGE_NAME = 'my-website'
        IMAGE_TAG = "v${BUILD_NUMBER}"
        CONTAINER_NAME = 'my-website-ci'
    }

    stages {
        stage('Check Docker') {
            steps {
                powershell '''
                    & $env:DOCKER --version
                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker CLI failed"
                    }

                    & $env:DOCKER info
                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker Engine unavailable"
                    }
                '''
            }
        }

        stage('Build Image') {
            steps {
                powershell '''
                    & $env:DOCKER build `
                        -t "$($env:IMAGE_NAME):$($env:IMAGE_TAG)" .

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker image build failed"
                    }
                '''
            }
        }

        stage('Smoke Test') {
            steps {
                powershell '''
                    & $env:DOCKER rm -f $env:CONTAINER_NAME 2>$null

                    & $env:DOCKER run -d `
                        -p 18080:80 `
                        --name $env:CONTAINER_NAME `
                        "$($env:IMAGE_NAME):$($env:IMAGE_TAG)"

                    if ($LASTEXITCODE -ne 0) {
                        throw "Container failed to start"
                    }

                    try {
                        $healthy = $false

                        for ($i = 0; $i -lt 10; $i++) {
                            try {
                                $response = Invoke-WebRequest `
                                -Uri "http://127.0.0.1:18080/" `
                                -UseBasicParsing `
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
                            throw "Website health check failed"
                        }

                        Write-Host "WEBSITE IS HEALTHY!"
                    }
                    finally {
                        & $env:DOCKER rm -f $env:CONTAINER_NAME
                    }
                '''
            }
        }
    }
}
