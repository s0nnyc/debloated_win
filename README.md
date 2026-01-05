# Minimalistic Win 10 Business. Used for games not properly running on Linux

## Download

Get a windows 10 Business 22H2 from massgrave.dev. I've tried LTSC versions, but they are almost impossible to activate and I ran into multiple problems when trying to install it from the stick

[https://massgrave.dev/windows_10_links](https://massgrave.dev/windows_10_links)

## Activate

Open PowerShell
1. Click the Start Menu, type PowerShell, then open it.

2. Copy and paste the code below, then press enter.

`irm https://get.activated.win | iex`

If the above is blocked (by ISP/DNS), try this (needs updated Windows 10 or 11):

`iex (curl.exe -s --doh-url https://1.1.1.1/dns-query https://get.activated.win | Out-String)`

Older versions of Windows will require running this command beforehand:

`[Net.ServicePointManager]::SecurityProtocol=[Net.SecurityProtocolType]::Tls12`

3. The activation menu will appear. Choose the green-highlighted options to activate Windows or Office.

4. **Done!**


## 0. Preconditions

- Fresh install
- Local admin account
- Internet connected only after step 3
- PowerShell as Administrator

## 1. AppX removal (core ballast)

Open PowerShell as administrator

```powershell
$remove = @(
"Microsoft.BingWeather",
"Microsoft.GetHelp",
"Microsoft.Getstarted",
"Microsoft.MicrosoftOfficeHub",
"Microsoft.MicrosoftSolitaireCollection",
"Microsoft.MixedReality.Portal",
"Microsoft.Office.OneNote",
"Microsoft.People",
"Microsoft.SkypeApp",
"Microsoft.Windows.Photos",
"Microsoft.WindowsAlarms",
"Microsoft.WindowsCamera",
"Microsoft.WindowsFeedbackHub",
"Microsoft.WindowsMaps",
"Microsoft.WindowsSoundRecorder",
"Microsoft.XboxApp",
"Microsoft.XboxGameOverlay",
"Microsoft.XboxGamingOverlay",
"Microsoft.XboxIdentityProvider",
"Microsoft.XboxSpeechToTextOverlay",
"Microsoft.YourPhone",
"Microsoft.ZuneMusic",
"Microsoft.ZuneVideo",
"Microsoft.BingSearch",
"Microsoft.Copilot"
)

foreach ($pkg in $remove) {
  Get-AppxPackage -AllUsers -Name $pkg | Remove-AppxPackage -AllUsers -ErrorAction SilentlyContinue
  Get-AppxProvisionedPackage -Online |
    Where-Object {$_.DisplayName -eq $pkg} |
    Remove-AppxProvisionedPackage -Online -ErrorAction SilentlyContinue
}
```
Verification (must be clean)
```powershell
Get-AppxPackage -AllUsers |
Where-Object {
  $_.Name -match "bing|xbox|zune|solitaire|skype|people|onenote|mixedreality|yourphone|feedback|copilot"
}
```
```powershell
Get-AppxProvisionedPackage -Online |
Where-Object {
  $_.DisplayName -match "bing|xbox|zune|solitaire|skype|people|onenote|mixedreality|yourphone|feedback|copilot"
}
```

### Note
You might get 1 or 2 errors - GetHelp and Pictures not getting deleted


## 2. OneDrive removal
```cmd
taskkill /f /im OneDrive.exe
%SystemRoot%\SysWOW64\OneDriveSetup.exe /uninstall
```
Then:

Delete C:\Users\<you>\OneDrive
Remove startup entry (Task Manager → Startup)

## 3. Services to disable (services.msc)

Set **Startup type = Disabled**:

- Connected User Experiences and Telemetry
- Diagnostic Policy Service
- Diagnostic Service Host
- Diagnostic System Host
- SysMain
- Windows Search (if no indexing)
- Xbox Accessory Management Service
- Xbox Live Auth Manager
- Xbox Live Game Save
- Xbox Networking Service
- Retail Demo Service
- Print Spooler (if no printer)
- Bluetooth Support Service (if unused)
- Downloaded Maps Manager


## 4. Scheduled tasks (Task Scheduler)

Disable **entire folders**:

- Microsoft → Windows → Application Experience
- Microsoft → Windows → Customer Experience Improvement Program
- Microsoft → Windows → Autochk
- Microsoft → Windows → DiskDiagnostic
- Microsoft → Windows → Feedback
- Microsoft → Windows → Maps
- Microsoft → Windows → Windows Error Reporting


## 5. Group Policy hardening (thru gpedit.msc)

## CONSUMER NOISE

### Disable consumer experiences
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Cloud Content
```

**Policy**  
Turn off Microsoft consumer experiences  

**Setting**  
Enabled  

---

### Advertising ID
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ System
      └─ User Profiles
```

**Policy**  
Turn off the advertising ID  

**Setting**  
Enabled  

---

### Tailored experiences
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Cloud Content
```

**Policy**  
Turn off tailored experiences  

**Setting**  
Enabled  

---

## CORTANA / SEARCH

### Allow Cortana
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Search
```

**Policy**  
Allow Cortana  

**Setting**  
Disabled  

---

### Web search (Bing in Start)
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Search
```

**Policies**  
- Do not allow web search  
- Don't search the web or display web results in Search  

**Setting**  
Both = Enabled  

---

### Search highlights
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Search
```

**Policy**  
Allow search highlights  

**Setting**  
Disabled  

---

## NEWS

### News and interests
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ News and interests
```

**Policy**  
Enable news and interests on the taskbar

**Setting**  
Disabled  

---

## WINDOWS UPDATE

### Configure Automatic Updates
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Windows Update
```

**Policy**  
Configure Automatic Updates  

**Setting**  
Enabled  

**Option**  
2 – Notify for download and auto install  

---

### Feature Updates deferral
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Windows Update
         └─ Windows Update for Business
```

**Policy**  
Select when Preview Builds and Feature Updates are received  

**Setting**  
Enabled  

**Options**  
- Defer feature updates: 365 days  

---

### Quality Updates deferral
**Path**
```
Computer Configuration
└─ Administrative Templates
   └─ Windows Components
      └─ Windows Update
         └─ Windows Update for Business
```

**Policy**  
Select when Quality Updates are received  

**Setting**  
Enabled  

**Options**  
- Defer quality updates: 30 days  


## 6. Defender (Business-safe reduction, not removal)

If Defender must stay:
```powershell
Set-MpPreference -DisableRealtimeMonitoring $false
Set-MpPreference -DisableCloudProtection $true
Set-MpPreference -SubmitSamplesConsent 2
Set-MpPreference -MAPSReporting 0
```
Optional exclustions for performance:
```powershell
Add-MpPreference -ExclusionPath "D:\Games"
```


## 7. UI + shell cleanup

- Settings → Personalization:
  - Transparency **Off**
  - Animations **Off**

Taskbar:
Search → Hidden
Widgets → Off
News & interests → Off

Start Menu:
Remove all tiles
Disable suggestions


## 8. Drivers discipline

Install only:

- Chipset
- GPU
- Network

Avoid:
- OEM control suites
- RGB software
- Audio enhancers unless required


## 9. Optional but recommended (clean ISO)

Use NTLite:

- Remove Store (policy-safe in Business)
- Remove Edge WebView (if no Store/UWP)
- Remove handwriting, speech, OCR
- Strip legacy media features
- Keep .NET, WMI, PowerShell


## Resulting system state

- Idle RAM: ~1.8–2.2 GB
- Zero consumer apps
- Minimal background telemetry
- Predictable updates
- Stable for dev, VM, gaming, tooling
- This is the maximum practical debloat for Windows 10 Business without breaking servicing or enterprise features.
