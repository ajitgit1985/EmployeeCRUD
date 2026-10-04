pipeline {
    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build --configuration Release --no-restore'
            }
        }

        stage('Test') {
            steps {
                bat 'dotnet test --configuration Release --no-build'
            }
        }

        stage('Publish') {
            steps {
                bat 'if exist "%WORKSPACE%\\publish" rmdir /s /q "%WORKSPACE%\\publish"'
                bat 'dotnet publish --configuration Release --no-build --output "%WORKSPACE%\\publish"'
            }
        }

        stage('Deploy to IIS') {
            steps {
                powershell '''
                    Import-Module WebAdministration

                    Write-Host "Stopping IIS application..."

                    Stop-Website -Name "myapp" -ErrorAction SilentlyContinue
                    Stop-WebAppPool -Name "myapp" -ErrorAction SilentlyContinue

                    Start-Sleep -Seconds 3

                    $source = "$env:WORKSPACE\\publish"
                    $destination = "C:\\inetpub\\wwwroot\\myapp"

                    Write-Host "Cleaning deployment folder..."

                    if (Test-Path $destination) {
                        Get-ChildItem $destination -Force |
                            Remove-Item -Recurse -Force
                    }
                    else {
                        New-Item -ItemType Directory -Path $destination -Force
                    }

                    Write-Host "Copying published application..."

                    Copy-Item "$source\\*" $destination -Recurse -Force

                    Write-Host "Starting IIS application..."

                    Start-WebAppPool -Name "myapp"
                    Start-Website -Name "myapp"

                    Write-Host "Deployment completed."
                '''
            }
        }

        stage('Health Check') {
            steps {
                powershell '''
                    Start-Sleep -Seconds 5

                    $response = Invoke-WebRequest `
                        -Uri "http://localhost/Employees" `
                        -UseBasicParsing

                    if ($response.StatusCode -ne 200) {
                        throw "Health check failed. HTTP Status: $($response.StatusCode)"
                    }

                    Write-Host "Health Check SUCCESS"
                    Write-Host "HTTP Status: $($response.StatusCode)"
                '''
            }
        }
    }

    post {
        success {
            echo 'EmployeeCRUD deployment SUCCESS'
        }

        failure {
            echo 'EmployeeCRUD deployment FAILED'
        }
    }
}