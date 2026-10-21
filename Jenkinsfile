pipeline { 
    agent any

tools {
    // Requires .NET SDK 6 to be configured in Jenkins
    // under Manage Jenkins → Tools.
    dotnetsdk 'dotnet-6'
}

stages {
    stage('Checkout Code') {
        steps {
            checkout scm
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