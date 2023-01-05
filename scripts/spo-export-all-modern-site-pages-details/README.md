---
plugin: add-to-gallery
---

# Export all modern site pages details from Site Pages library

## Summary

This sample will export the required modern site pages information to CSV.

## Implementation

- Open Windows PowerShell ISE
- Create a new file
- Write a script as below,
- We will connect to the site from which site we want to get modern pages. And then will fetch modern site pages from the pages library and then will export them to CSV.

# [PnP PowerShell](#tab/pnpps)
```powershell
$siteURL = "https://domain.sharepoint.com/sites/SPFxLearning/"
$username = "chandani@domain.onmicrosoft.com"
$password = "********"
$secureStringPwd = $password | ConvertTo-SecureString -AsPlainText -Force 
$creds = New-Object System.Management.Automation.PSCredential -ArgumentList $username, $secureStringPwd
$dateTime = "_{0:MM_dd_yy}_{0:HH_mm_ss}" -f (Get-Date)
$basePath = "E:\Contribution\PnP-Scripts\Logs\"
$csvPath = $basePath + "\ModernSitePages" + $dateTime + ".csv"
$global:modernSitePages = @()

Function Login() {
    [cmdletbinding()]
    param([parameter(Mandatory = $true, ValueFromPipeline = $true)] $creds)     
    Write-Host "Connecting to Site...'$($siteURL)'" -f Yellow   
    Connect-PnPOnline -Url $siteURL -Credential $creds
    Write-Host "Connection Successful!" -f Green 
}

Function GetModernSitePages {    
    try {
        Write-Host "Getting modern site pages information..."  -ForegroundColor Yellow 
        
        $sitePages = Get-PnPListItem -List "Site Pages" -Query "<View><Query><Where><Eq><FieldRef Name='ContentTypeId'/><Value Type='Text'>0x0101009D1CB255DA76424F860D91F20E6C411800E53BD99D3E0D8545813DF21536D1B228</Value></Eq></Where></Query></View>"
        if ($sitePages.count) {
            ForEach ($Page in $sitePages) {
                $sitePagesInfo = New-Object PSObject -Property ([Ordered] @{
                        'ID'               = $Page.ID
                        'Title'            = $Page.FieldValues.Title
                        'Description'      = $Page.FieldValues.Description
                        'Page Layout Type' = $Page.FieldValues.PageLayoutType
                        'FileRef'          = $Page.FieldValues.FileRef  
                        'FileLeafRef'      = $Page.FieldValues.FileLeafRef      
                        'Created'          = Get-Date -Date $Page.FieldValues.Created_x0020_Date -Format "dddd MM/dd/yyyy HH:mm"
                        'Modified'         = Get-Date -Date $Page.FieldValues.Last_x0020_Modified -Format "dddd MM/dd/yyyy HH:mm"
                        'Modified By'      = $Page.FieldValues.Modified_x0020_By
                        'Created By'       = $Page.FieldValues.Created_x0020_By
                        'Author'           = $Page.FieldValues.Author.Email
                        'Editor'           = $Page.FieldValues.Editor.Email
                        'BannerImage Url'  = $Page.FieldValues.BannerImageUrl.Url   
                        'File_x0020_Type'  = $Page.FieldValues.File_x0020_Type   
                    })
                $global:modernSitePages += $sitePagesInfo
            }
        }
        else {
            Write-Host "Modern site pages not found in ${siteURL}" -ForegroundColor Gray
        }        
        Write-Host "Getting modern site pages successfully!"  -ForegroundColor Green 
    }
    catch {
        Write-Host "Error in getting modern site pages:" $_.Exception.Message -ForegroundColor Red                 
    }
    Write-Host "Exporting to CSV..."  -ForegroundColor Yellow 
    $global:modernSitePages | Export-Csv $csvPath -NoTypeInformation -Append
    Write-Host "Exported to CSV successfully!"  -ForegroundColor Green	

}

Function StartProcessing {
    Login($creds);
    GetModernSitePages
}

StartProcessing
```
[!INCLUDE [More about PnP PowerShell](../../docfx/includes/MORE-PNPPS.md)]
***


## Contributors

| Author(s) |
|-----------|
| Chandani Prajapati (https://github.com/chandaniprajapati) |

[!INCLUDE [DISCLAIMER](../../docfx/includes/DISCLAIMER.md)]
<img src="https://pnptelemetry.azurewebsites.net/script-samples/scripts/spo-export-all-modern-site-pages-details" aria-hidden="true" />
