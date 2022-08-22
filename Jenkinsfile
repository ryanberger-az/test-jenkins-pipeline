pipeline {
  agent {
    node {
      label 'win19-cycleready'
    }

  }
  stages {
    stage('Prepare Windows Node') {
      steps {
        powershell """
                        Write-Output \'Agent Information\'
                        Write-Output \'Jenkins Workspace is set to: ${WORKSPACE}\'
                        Write-Output \'Running on node: ${NODE_NAME}\'
                        Write-Output \'Running for branch: ${BRANCH_NAME}\'
                        Write-Output \' \'
                        Write-Output \'Node IP Address:\'
                        Get-netIPAddress -PrefixOrigin "Dhcp" -AddressFamily "IPv4" | Format-Table
                        """
        powershell '''
        Write-Output \'Updating Windows Dependencies\'

        Write-Output \'Updating SecurityProtocolType\'
        [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
        
        Write-Output \'Install NuGet Base\'
        Install-PackageProvider -Name NuGet -Force

        Write-Output \'Update PowershellGet\'
        Install-Module PowershellGet -Force
        Update-Module -Name PowerShellGet

        Write-Output \'Install NuGet v2\'
        Register-PackageSource -Name MyNuGet -Location https://www.nuget.org/api/v2 -ProviderName NuGet -Trusted
        '''
        powershell """
                        Write-Output \'Install Browsers and Web Drivers\'

                        Write-Output \'Installing Chocolatey Chrome + Edge Browsers\'
                        choco install googlechrome --y --ignore-checksums
                        choco install microsoft-edge --y --ignore-checksums

                        Write-Output \'Creating directory for web drivers\'
                        New-Item -Path "C:\\" -Name "tools" -ItemType "directory" -Force
                        New-Item -Path "C:\\tools\\" -Name "selenium" -ItemType "directory" -Force

                        Write-Output \'Installing ChromeDriver\'
                        \$ChromeDriverPackage = Install-Package Selenium.WebDriver.ChromeDriver -Destination . -Force
                        \$ChromeDriverPackageExe = -join(\$ChromeDriverPackage.Name,".",\$ChromeDriverPackage.Version, "\\driver\\win32\\chromedriver.exe")
                        Copy-Item \$ChromeDriverPackageExe -Destination "C:\\tools\\selenium" -Force

                        Write-Output \'Installing IEDriver\'
                        \$IEDriverPackage = Install-Package Selenium.WebDriver.IEDriver -Destination . -Force
                        \$IEDriverPackageExe = -join(\$IEDriverPackage.Name,".",\$IEDriverPackage.Version, "\\driver\\IEDriverServer.exe")
                        Copy-Item \$IEDriverPackageExe -Destination "C:\\tools\\selenium" -Force

                        Write-Output \'Installing MSEdgeDriver\'
                        \$MSEdgeDriverPackage = Install-Package Selenium.WebDriver.MSEdgeDriver -Destination . -Force
                        \$MSEdgeDriverPackageExe = -join(\$MSEdgeDriverPackage.Name,".",\$MSEdgeDriverPackage.Version, "\\driver\\win32\\msedgedriver.exe")
                        Copy-Item \$MSEdgeDriverPackageExe -Destination "C:\\tools\\selenium" -Force
                        ls C:\\tools\\selenium

                        Write-Output \'Installing WinAppDriver and checking install\'
                        New-Item -Path "${WORKSPACE}\\" -Name "temp" -ItemType "directory"
                        Invoke-WebRequest 'https://github.com/microsoft/WinAppDriver/releases/download/v1.2.1/WindowsApplicationDriver_1.2.1.msi' -Method 'GET' -OutFile 'WindowsApplicationDriver.msi'
                        Start-Process msiexec.exe -Wait -ArgumentList \'/i ${WORKSPACE}\\WindowsApplicationDriver.msi /QN /L*V ${WORKSPACE}\\temp\\msilog.log\'
                        cd "C:\\Program Files (x86)\\Windows Application Driver"
                        ls
                        """
  }
    }
  environment {
    CI_REPO_NAME = 'test-jenkins-pipeline'
    CI_REPO_BRANCH = 'master'
    VMAGENT_LABEL = 'win19-cycleready'
    CI_CYCLE_PROJECT = 'appliance-browser-testing'
    CI_CYCLE_PROJECT_FILE = 'appliance-browser-testing.cycproj'
    CI_CYCLE_USER = 'cyclejenkinsuser'
    MAX = 24
    RANDOM_NUM = "${Math.abs(new Random().nextInt(MAX+1))}"
  }
  triggers {
    cron('2 * * * *')
  }
}
}