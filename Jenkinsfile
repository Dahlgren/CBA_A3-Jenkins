pipeline {
  agent {
    label 'mikero'
  }

  stages {
    stage('Checkout') {
      steps {
        dir('x/cba') {
          script {
            def cbaGit = git url: 'https://github.com/CBATeam/CBA_A3.git', branch: 'master', changelog: false, poll: true
            env.CBA_A3_COMMIT = cbaGit.GIT_COMMIT
          }

          // Fix legacy pboproject parameter 
          powershell '((Get-Content -path tools/make.py -Raw) -replace \'"\\+X"\', \'"+G"\') | Set-Content -Path tools/make.py'
          powershell '((Get-Content -path tools/make.py -Raw) -replace \'"-X"\', \'"-G"\') | Set-Content -Path tools/make.py'

          // Set bad exit code on error
          powershell '((Get-Content -path tools/make.py -Raw) -replace \'sys.exit\\(0\\)\', \'sys.exit(len(failedBuilds))\') | Set-Content -Path tools/make.py'
        }
      }
    }

    stage('Python') {
      steps {
        bat 'curl https://www.python.org/ftp/python/3.7.4/python-3.7.4-embed-win32.zip --output python3.zip'
        powershell 'Expand-Archive -Path python3.zip -DestinationPath python3'
      }
    }

    stage('Arma Data') {
      steps {
        // Link Arma 3 Data
        bat 'mklink /j a3 %A3_DATA%\\a3'
      }
    }

    stage('Build') {
      steps {
        // Set correct excludes for CBA A3 in pboproject
        bat 'regedit /S %WORKSPACE%\\pboproject_cba_a3.reg'

        // Mount P: drive
        bat 'subst P: .'

        // Build CBA A3 with CI exit status and external files check enabled
        bat 'python3\\python.exe x/cba/tools/make.py ci checkexternal'

        // Move built mod to root of workspace
        bat 'move x/cba/release/@cba_a3 @cba_a3'
      }
    }

    stage('Steam Workshop') {
      steps {
        publishSteamWorkshop '1920404324', '@cba_a3', "https://github.com/CBATeam/CBA_A3/commit/${env.CBA_A3_COMMIT}"
      }
    }
  }

  post { 
    always {
      // Restore default excludes in pboproject
      bat 'regedit /S %WORKSPACE%\\pboproject_default.reg'

      // Archive built mod on success
      archiveArtifacts allowEmptyArchive: true, artifacts: '@cba_a3/**/*'

      // Archive pboproject log files
      archiveArtifacts allowEmptyArchive: true, artifacts: 'temp/*.log'

      // Cleanup workspace to avoid wasting disk space
      deleteDir()

      // Dismount P: drive
      bat 'subst P: /D'
    }
  }
}

void publishSteamWorkshop(String id, String mod, String changeNote) {
  bat "\"C:\\Program Files (x86)\\Steam\\SteamApps\\common\\Arma 3 Tools\\Publisher\\PublisherCmd.exe\" update /changeNote:$changeNote /id:$id /path:$mod"
}
