
pipeline {
    agent any

    stages {
        stage('Checkout Code') {
            steps {
                checkout scm
            }
        }

        stage('Install .NET 6 SDK') {
            steps {
                powershell '''
                    $installDir = Join-Path $env:WORKSPACE ".dotnet"
                    $dotnetExe = Join-Path $installDir "dotnet.exe"

                    New-Item -ItemType Directory -Force `
                        -Path $installDir | Out-Null

                    $sdkInstalled = $false

                    if (Test-Path $dotnetExe) {
                        $sdks = & $dotnetExe --list-sdks
                        if ($sdks -match '^6\\.0\\.') {
                            $sdkInstalled = $true
                        }
                    }

                    if (-not $sdkInstalled) {
                        Write-Host "Installing .NET 6 SDK..."

                        $installer = Join-Path $env:TEMP "dotnet-install.ps1"

                        Invoke-WebRequest `
                            -Uri "https://dot.net/v1/dotnet-install.ps1" `
                            -OutFile $installer

                        & $installer -Channel 6.0 -InstallDir $installDir

                        if ($LASTEXITCODE -ne 0) {
                            throw ".NET 6 SDK installation failed"
                        }
                    }

                    & $dotnetExe --list-sdks

                    if ($LASTEXITCODE -ne 0) {
                        throw ".NET SDK verification failed"
                    }
                '''
            }
        }

        stage('Setup .NET') {
            steps {
                bat '"%WORKSPACE%\\.dotnet\\dotnet.exe" --version'
            }
        }

        stage('Restore Dependencies') {
            steps {
                bat '"%WORKSPACE%\\.dotnet\\dotnet.exe" restore'
            }
        }

        stage('Build') {
            steps {
                bat '"%WORKSPACE%\\.dotnet\\dotnet.exe" build --no-restore'
            }
        }

        stage('Test') {
            steps {
                bat '"%WORKSPACE%\\.dotnet\\dotnet.exe" test --no-build --verbosity normal'
            }
        }
    }
}
