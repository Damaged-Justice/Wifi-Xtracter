# Wifi-Xtracter is a simple executable to automate the process of viewing cached wifi passwords within a computer. utilizing the netsh command and proper appending syntax. 

# PowerShell script to extract and save Wi-Fi passwords

# Display ASCII Art
$asciiArt = @"
  __        ___  _  _  ____  ____  ____  __  __  ____  __    ___  __ _ 
 / _\      / __)/ )( \(  __)(  __)/ ___)(  )(  )(  _ \(  )  / __)(  / )
/    \    ( (__ ) \/ ( ) _)  ) _) \___ \ )( /)(_)( )   / )(__( (__ )  (
\_/\_/     \___)\____/(____)(__)  (____/(__)\____/(__\_)(____/\__\_)
                           WIFI-Xtracter
"@

Write-Host $asciiArt

# Function to extract passwords
Function Get-WiFiPasswords {
    [cmdletbinding()]
    Param()

    $wifiProfiles = netsh wlan show profiles | Select-String -Pattern "All User Profile\s*:\s*(.+)" | ForEach-Object {
        $_.Matches.Groups[1].Value.Trim()
    }

    $wifiPasswords = @()

    foreach ($profile in $wifiProfiles) {
        $wifiPassword = netsh wlan show profile name="$profile" key=clear | Select-String -Pattern "Key Content\s*:\s*(.+)" | ForEach-Object {
            $_.Matches.Groups[1].Value.Trim()
        }

        $wifiPasswords += [PSCustomObject]@{
            SSID = $profile
            Password = $wifiPassword
        }
    }

    return $wifiPasswords
}

# Save Wi-Fi passwords to a file on Desktop
$desktopPath = [Environment]::GetFolderPath("Desktop")
$outputFile = Join-Path $desktopPath "WiFiPasswords.txt"
Get-WiFiPasswords | Format-Table -AutoSize | Out-String -Width 4096 | Out-File $outputFile

Write-Host "Wi-Fi passwords saved to $outputFile"
