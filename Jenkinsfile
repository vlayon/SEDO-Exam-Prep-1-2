
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
            $ErrorActionPreference = "Stop"

            $installDir = Join-Path $env:WORKSPACE ".dotnet"
            $dotnetExe = Join-Path $installDir "dotnet.exe"

            New-Item -ItemType Directory -Force `
                -Path $installDir | Out-Null

            $sdkInstalled = $false

            if (Test-Path $dotnetExe) {
                $sdks = & $dotnetExe --list-sdks

                if ($sdks -match '^6\\.0\\.') {
                    $sdkInstalled = $true
                    Write-Host ".NET 6 SDK is already installed."
                }
            }

            if (-not $sdkInstalled) {
                Write-Host "Installing .NET 6 SDK..."

                $installer = Join-Path $env:TEMP "dotnet-install.ps1"

                Invoke-WebRequest `
                    -Uri "https://dot.net/v1/dotnet-install.ps1" `
                    -OutFile $installer

                & $installer -Channel 6.0 -InstallDir $installDir

                # Verify the actual installation rather than relying
                # on the installer's exit-code variable.
                if (!(Test-Path $dotnetExe)) {
                    throw ".NET installation failed: dotnet.exe not found."
                }

                $sdks = & $dotnetExe --list-sdks

                if ($sdks -notmatch '^6\\.0\\.') {
                    throw ".NET 6 SDK was not found after installation."
                }
            }

            Write-Host "Installed SDKs:"
            & $dotnetExe --list-sdks

            Write-Host ".NET SDK verification successful."
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
