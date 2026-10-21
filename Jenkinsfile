pipeline {
    agent any

    environment {
        DOTNET_ROOT = "${WORKSPACE}/.dotnet"
        "PATH+DOTNET" = "${WORKSPACE}/.dotnet"
    }

    stages {
        stage('Build and Test') {
            when {
                anyOf {
                    branch 'main'
                    branch 'feature'
                }
            }

            stages {
                stage('Checkout Code') {
                    steps {
                        checkout scm
                    }
                }

                stage('Install .NET 6 SDK') {
                    steps {
                        powershell '''
                            $installDir = "$env:WORKSPACE/.dotnet"
                            $dotnetExe = "$installDir/dotnet.exe"

                            if (!(Test-Path $dotnetExe)) {
                                Write-Host "Installing .NET 6 SDK..."

                                New-Item -ItemType Directory `
                                    -Force -Path $installDir | Out-Null

                                $installer = "$env:TEMP/dotnet-install.ps1"

                                Invoke-WebRequest `
                                    -Uri "https://dot.net/v1/dotnet-install.ps1" `
                                    -OutFile $installer

                                & $installer `
                                    -Channel 6.0 `
                                    -InstallDir $installDir

                                if ($LASTEXITCODE -ne 0) {
                                    throw ".NET SDK installation failed"
                                }
                            } else {
                                Write-Host ".NET SDK already installed."
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
                        bat 'dotnet --version'
                    }
                }

                stage('Restore Dependencies') {
                    steps {
                        bat 'dotnet restore'
                    }
                }

                stage('Build') {
                    steps {
                        bat 'dotnet build --no-restore'
                    }
                }

                stage('Test') {
                    steps {
                        bat 'dotnet test --no-build --verbosity normal'
                    }
                }
            }
        }
    }
}