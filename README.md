# USING VIRTUALENVWRAPPER'S WORKON COMMAND IN POWERSHELL
Replicating virtualenvwrapper commands in Powershell.

The popular Pythonic virtualenvwrapper utitlity works way better in Ubuntu than Windows.

Namely, if you install virtualenvwrapper-win...

    pip install virtualenvwrapper-win

...the mkvirtualenv and rmvirtualenv commands behave as expected, but some of the *workon* commands are funky or don't work at all.

Personally, I've never fully gotten used to the *venv* command in Python, and still prefer using pip and virtualenv to conda. 

Of course, you could simply develop in a containerized environment and bypass all of this mishagos.

But sometimes I like to just dig in and rely on old habits...

(I know: I'm a dinosaur.)

As such, in Windows, I really miss the *workon* commands--because they are super useful for swapping between environments speedily...again, if you're stubbornly gonna use virtualenvs instead of conda's environment and package management.

To get around this limitation and to replicate all the old school Python plus Ubuntu feels, place the script below in your Powershell profile.

The script assumes you have virtualenvwrapper-win installed and all the proper env vars set correctly.

If virtualenvwrapper-win is config'ed properly, this script will replicate the following virtualenvwrapper commands:

- workon : lists all environemnts
- workon <environment_name> : activates the environment
- workoncd <environment_name> : activates the environment and changes directory to the project directory if and only if you have already set the project directory via the *setprojectdir* command per virtualenvwrapper.

FWIW, the simplest way to set the project directory is to cd to the project directory and execute...

    setprojectdir .

To edit your Powershell profile, you can evoke notepad from directly within a terminal in Powershell...

    notepad $PROFILE

Then cut and paste in the script below. Be sure to replace *username* with your actual username.

FYI: The script assumes your WORKON_HOME env var is set to C:\Users\username\Envs

Be sure to save your changes.

Finally, reload the Poewrshell profile with...

    . $PROFILE

Now *workon* will function just like it does on Ubuntu. (Albeit you'll have the additional *workoncd* command.)

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

