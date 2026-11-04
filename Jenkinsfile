pipeline {
    agent any

    stages {
        stage('Diagnose Docker') {
            steps {
                powershell '''
                    $docker = 'C:\\Users\\kc373\\AppData\\Local\\Programs\\DockerDesktop\\resources\\bin\\docker.exe'

                    Write-Host "STEP 1: Checking Docker executable"

                    if (!(Test-Path $docker)) {
                        throw "Docker executable not found!"
                    }

                    Write-Host "STEP 2: Docker executable found"

                    & $docker --version

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker CLI failed"
                    }

                    Write-Host "STEP 3: Checking Docker Engine"

                    & $docker info

                    if ($LASTEXITCODE -ne 0) {
                        throw "Docker Engine access failed"
                    }

                    Write-Host "SUCCESS: Jenkins can access Docker!"
                '''
            }
        }
    }
}
