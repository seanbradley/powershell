# USING WORKON IN POWERSHELL
Replicating virtualenvwrapper commands in Powershell.

The popular virtualenvwrapper library works way better in Ubuntu than Windows.

Namely, even if you install virtualenvwrapper-win...

    pip install virtualenvwrapper-win

...some of the *workon* commands are funky or don't work at all in Windows.

To get around this, place the script below in your Powershell profile.

The script assumes you have virtualenvwrapper-win installed and all the proper env vars set correctly.

If virtualenvwrapper-win is config'ed properly, this script will replicate the following virtualenvwrapper commands:

- workon : lists all environemnts
- workon <environment_name> : activates the environment
- workoncd <environment_name> : activates the environment and changes directory to the project directory if and only if you have already set the project directory via the *setprojectdir* command per virtualenvwrapper.

The simplest way to set the project directory is to cd to the project directory and execute...

    setprojectdir .

To leverage this Powershell profile script, you can edit the profile by evoking notepad from within a terminal in Powershell...

    notepad $PROFILE

Then cut and paste the script below; be sure to replace *username* with your actual username.

(The script assumes your WORKON_HOME env var is set to C:\Users\username )

Then save your changes.

Finally, reload the profile with...

    . $PROFILE

---

	# Ensure WORKON_HOME environment variable is set
	$env:WORKON_HOME = "C:\Users\<username>\Envs"

	# Function to list or activate virtual environments
	function workon {
		param([string]$environment = "")
		
		# List all environments if no specific environment is provided
		if (-not $environment) {
			Write-Host "Available virtual environments:"
			Get-ChildItem -Path $env:WORKON_HOME -Directory | ForEach-Object { Write-Host $_.Name }
		}
		else {
			$activateScriptPath = Join-Path -Path $env:WORKON_HOME -ChildPath "$environment\Scripts\activate.ps1"
			
			if (Test-Path $activateScriptPath) {
				& $activateScriptPath
			}
			else {
				Write-Host "Virtual environment '$environment' not found."
			}
		}
	}

	function Activate-EnvAndCd {
		param([string]$envName)
		
		# Invoke the workon function to activate the environment
		workon $envName
		
		# Only proceed if an environment name is provided
		if (-not $envName) {
			return
		}
		
		# Construct the path to the project directory file
		$projectDirPathFile = Join-Path -Path $env:WORKON_HOME -ChildPath "$envName\Lib\site-packages\virtualenv_path_extensions.pth"
		
		# Check if the project directory file exists and read the first line
		if (Test-Path $projectDirPathFile) {
			$projectDir = Get-Content $projectDirPathFile | Select-Object -First 1
			if ($projectDir -and (Test-Path $projectDir)) {
				Write-Host "Changing directory to project: $projectDir"
				Set-Location -Path $projectDir
			}
			else {
				Write-Host "The project directory has not been set via setprojectdir, or the path is invalid."
			}
		}
		else {
			Write-Host "The project directory has not been set via setprojectdir."
		}
	}


	# Alias for Activate-EnvAndCd to use with 'workoncd'
	Set-Alias -Name workoncd -Value Activate-EnvAndCd

