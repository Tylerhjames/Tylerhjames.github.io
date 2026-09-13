#Requires -Version 5.1
<#
Short GitHub Pages entry point for the public scripts-only v0.3.4-rc2 release.
Fetching this script with `irm ... | iex` executes the served script before an
independent hash check. Use Remote-Test-Verify-rc2.md for a hash-pinned first test.
The loader and package downloaded by this script are SHA256 checked.
#>
param([switch]$Verify)
$ErrorActionPreference = 'Stop'
$loaderUri = 'https://github.com/Tylerhjames/cbit-customer-toolkit/releases/download/v0.3.4-rc2/Get-CbitToolkit.ps1'
$loaderSha256 = 'C57ADD8C40E0B3EDE06B5EA3C540A2D57081FCAE336675EB9EC28430B733C748'
$packageUri = 'https://github.com/Tylerhjames/cbit-customer-toolkit/releases/download/v0.3.4-rc2/CBIT-CustomerToolkit-0.3.4.zip'
$packageSha256 = 'D5E6F243DF5B21FD4766890FEA051D2B5C7210B492EF684C94AA9CFEA15F685F'
$packageFolder = 'CBIT-CustomerToolkit-0.3.4'
$loaderPath = Join-Path ([IO.Path]::GetTempPath()) ('CBIT-loader-' + [guid]::NewGuid().ToString('N') + '.ps1')
try {
    Invoke-WebRequest -UseBasicParsing -Uri $loaderUri -OutFile $loaderPath -TimeoutSec 60 -MaximumRedirection 5 -ErrorAction Stop
    if ((Get-FileHash -LiteralPath $loaderPath -Algorithm SHA256 -ErrorAction Stop).Hash -ine $loaderSha256) {
        throw 'Launcher integrity check failed. The toolkit was not started.'
    }
    & $loaderPath -PackageUri $packageUri -PackageSha256 $packageSha256 -PackageFolder $packageFolder -Verify:$Verify
}
finally {
    if (Test-Path -LiteralPath $loaderPath) { Remove-Item -LiteralPath $loaderPath -Force }
}
