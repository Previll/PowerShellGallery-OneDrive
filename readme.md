# OneDrive PowerShell Module

The OneDrive PowerShell module is available via PowerShellGallery.com. If you want to support and work with me feel free to make changes cloning this repo, change and send me and a pull request.

This OneDrive version (2.0.0 and higher in PowerShellGallery.com) supports:

- [x] OneDrive personal
- [x] OneDrive for Business

## Installation

Open PowerShell and

```powershell
Install-Module -Name OneDrive -Scope CurrentUser -force
```

You can update the module to a newer version with the same command (-force). If you don’t use PowerShellGet currently, go to the Gallery on https://www.powershellgallery.com/packages/OneDrive and click "Get Started".

Check your installation with

```powershell
Get-Help -Name OneDrive
```

## Authentication

Before you start using the OneDrive module you have register your script/application. This differs depending on the OneDrive version to be used.

### OneDrive Personal

Read this on my blog: https://www.sepago.de/blog/onedrive-powershell-module-new-version-with-improved-authentication/

- Go to: https://apps.dev.microsoft.com and login with your Microsoft Account (MSA). "Add an app" in the category "converged applications" 
  ![](https://www.sepago.de/wp-content/uploads/2017/12/oda01.png)

- Enter a name and press "create"
  ![](https://www.sepago.de/wp-content/uploads/2017/12/oda02.png)

- Press "Generate New Password" and save the password (app key)
  ![](https://www.sepago.de/wp-content/uploads/2017/12/oda03.png)

- Also save the "Application id"

- Press "Add Platforms" and select "Web"
  ![](https://www.sepago.de/wp-content/uploads/2017/12/oda04.png)

- Check "Allow implicit Flow" and enter a "Redirect URL". This is not a real URL. Choose a localhost address and note it. In my case I chose: http://localhost/login
  ![](https://www.sepago.de/wp-content/uploads/2017/12/oda05.png)
  ![](https://www.sepago.de/wp-content/uploads/2017/12/oda06.png)
  
- Press "Save"

- Now you have all necessary data for your app / script:

  - Client Id: 5dd40b03-0ead-451b-b5e3-f704550e8cca
  - AppKey: xqacs8K92MuCJKgciRHQ1Cf
  - RedirectURI: http://localhost/login

- To get an authentication token use: 

  ```powershell
  $Auth=Get-ODAuthentication -ClientID 5dd40b03-0ead-451b-b5e3-f704550e8cca -AppKey xqacs8K92MuCJKgciRHQ1Cf -RedirectURI http://localhost/login
  ```


  ![](https://www.sepago.de/wp-content/uploads/2017/12/oda07.png)


### OneDrive for Business

To use OneDrive for business you have to register your script/app to in Azure Active Directory

- Add an application in Azure Active Directory inside the Azure portal: https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/RegisteredApps

- Chose "New application registration"

- Give your application a name and an unique sign-on URL. The sign-on URL has to be a valid URL but doesn't have to exist. E.g.: http://sepago.de/1Drive4Business (make later sure that this url is in the reply url list of your application)

  ![](https://www.sepago.de/wp-content/uploads/2018/01/od4b01.png)

- Within the "Required permissions" add "Office 365 SharePoint Online (Microsoft.Sharepoint)"
  ![](https://www.sepago.de/wp-content/uploads/2018/01/od4b02.png)

- Select "Read and write user files"  below "delegated permissions" for the Office 365 API
  ![](https://www.sepago.de/wp-content/uploads/2018/01/od4b03.png)

- Generate a secrete key for this application and save it for later use. Also save the application Id
  ![](https://www.sepago.de/wp-content/uploads/2018/01/od4b04.png)
  ![](https://www.sepago.de/wp-content/uploads/2018/01/od4b05_0.png)

- You should now have the following parameter:

  - Client Id: 2831fc52-e1b8-4493-9f3a-a3dad74b2081
  - AppKey: TqoSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=
  - RedirectURI: http://sepago.de/1Drive4Business

- Additionally you need the resource URL for OneDrive for Business. Normally: https://<tenant>-my.sharepoint.com/. In our company this is the URL "https://sepagogmbh-my.sharepoint.com/" (the last one / is important).
  - Resource ID: https://sepagogmbh-my.sharepoint.com/

- To get an authentication token use: 

  ```powershell
  $Auth=Get-ODAuthentication -ClientId "2831fc52-e1b8-4493-9f3a-a3dad74b2081" -AppKey "TqoSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX="  -RedirectURI "http://sepago.de/1Drive4Business" -ResourceId "https://sepagogmbh-my.sharepoint.com/"
  ```

### Renew the authentication with a refresh token

An access token is 1 hour valid. You can get a new access token with the refresh token provided by the last authentication. This is necessary if you are creating a script that will work for a long time without further user input. Renew your access token automatically in the program code.

```powershell
$Auth=Get-ODAuthentication -ClientId 2831fc52-e1b8-4493-9f3a-a3dad74b2081 -AppKey "TqoSXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX=" -RedirectURI "http://sepago.de/1Drive4Business" -ResourceId "https://sepagogmbh-my.sharepoint.com/" -RefreshToken $LastAuth.refresh_token
```

- Where $LastAuth is your last authentication result (containing the refresh token)
- For OneDrive personal leave the ResourceId empty (-ResourceId "")

## Working with files and folders

Get an authentication code from above and store it in $Auth

### List files and folders

```powershell
Get-ODChildItems -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/" -path "/"
```

### List files and folders

```powershell
Remove-ODItem -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/" -path "/Upload"
```

### Creating a folder

```powershell
New-ODFolder -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/" -path "/" -FolderName "Upload"
```

### Upload local files to OneDrive

```powershell
Add-ODItem -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/" -LocalFile "D:\DEV\PowerShell\PowerShellGallery-OneDrive\Test\Uploads\IoT Workshop.pptx" -Path "/Upload" 
```

### List OneDrive drives

```powershell
Get-ODDrives -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/"
```

### Downloading some files

```powershell
Get-ODItem -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/" -Path "/Upload/Doings.txt" -LocalPath "D:\DEV\PowerShell\PowerShellGallery-OneDrive\Test\Downloads"
```

```powershell
Get-ODItem -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/" -Path "/Upload/Doings.txt" -LocalPath "D:\DEV\PowerShell\PowerShellGallery-OneDrive\Test\Downloads" -LocalFileName "Copy from OneDrive.Doings.txt"
```

### Delete a file in OneDrive

```powershell
Remove-ODItem -AccessToken $Auth.access_token -ResourceId "https://sepagogmbh-my.sharepoint.com/" -Path "/Upload/Doings.txt"
```

## Remarks

Links:

- My blog: https://www.sepago.de/blog/author/marcel-meurer/
- Linked-in:  https://www.linkedin.com/in/marcel-meurer-15b46b98/
- Twitter: https://twitter.com/MarcelMeurer
- About me / imprint: https://about.me/marcel.meurer



Feel free to contribute. If you want to stay informed follow this GitHub repo and me on Twitter: https://twitter.com/MarcelMeurer