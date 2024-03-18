# MS365GroupCreators
Functional new version of mggraph PowerShell script for MS365 group creators restriction. 

OLD SCRIPT: 

```Powershell
Import-Module Microsoft.Graph.Beta.Identity.DirectoryManagement
Import-Module Microsoft.Graph.Beta.Groups

Connect-MgGraph -Scopes "Directory.ReadWrite.All", "Group.Read.All"

$GroupName = "SomeGroup"
$AllowGroupCreation = "False"

$settingsObjectID = (Get-MgBetaDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).id

if(!$settingsObjectID)
{
    $params = @{
	  templateId = "62375ab9-6b52-47ed-826b-58e47e0e304b"
	  values = @(
		    @{
			       name = "EnableMSStandardBlockedWords"
			       value = "true"
		     }
	 	     )
	     }
	
    New-MgBetaDirectorySetting -BodyParameter $params
	
    $settingsObjectID = (Get-MgBetaDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).Id
}
 
$groupId = (Get-MgBetaGroup | Where-object {$_.displayname -eq $GroupName}).Id

$params = @{
	templateId = "62375ab9-6b52-47ed-826b-58e47e0e304b"
	values = @(
		@{
			name = "EnableGroupCreation"
			value = $AllowGroupCreation
		}
		@{
			name = "GroupCreationAllowedGroupId"
			value = $groupId
		}
	)
}

Update-MgBetaDirectorySetting -DirectorySettingId $settingsObjectID -BodyParameter $params

(Get-MgBetaDirectorySetting -DirectorySettingId $settingsObjectID).Values
```

The previous script uses a hash table with a key-value pair contained within the first "@params", that invokes the EnableMSStandardBlockedWords function, which is deprecated.

I have deleted that line, and only left the "templateID" in order to have a reference in case that the group specified in "$GroupName" does not exist.

Then, the main issue is that, if you run this script, the output won't never show the SomeGroup ID since is not going to be able to read it.

The cmdlet "Get-MgBetaGroup" can only return up to 100 element by default, so, is not going to properly associate the restriction to the "SomeGroup" because it won't be able to assign any value to $groupId showing a blank value in GroupCreationAllowedGroupId.

To correct this. an -All parameter must be indicated, in order to retrieve all element in the group list to be read.

NEW SCRIPT:

```PowerShell
Import-Module Microsoft.Graph.Beta.Identity.DirectoryManagement
Import-Module Microsoft.Graph.Beta.Groups

Connect-MgGraph -Scopes "Directory.ReadWrite.All", "Group.Read.All"

$GroupName = "SomeGroup"
$AllowGroupCreation = "False"

$settingsObjectID = (Get-MgBetaDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).id

if(!$settingsObjectID)
{
    $params = @{
	  templateId = "62375ab9-6b52-47ed-826b-58e47e0e304b"
	     }
	
    New-MgBetaDirectorySetting -BodyParameter $params
	
    $settingsObjectID = (Get-MgBetaDirectorySetting | Where-object -Property Displayname -Value "Group.Unified" -EQ).Id
}

$groups = Get-MgBetaGroup -All
 
$groupId = ($groups | Where-object {$_.displayname -eq $GroupName}).Id

$params = @{
	templateId = "62375ab9-6b52-47ed-826b-58e47e0e304b"
	values = @(
		@{
			name = "EnableGroupCreation"
			value = $AllowGroupCreation
		}
		@{
			name = "GroupCreationAllowedGroupId"
			value = $groupId
		}
	)
}

Update-MgBetaDirectorySetting -DirectorySettingId $settingsObjectID -BodyParameter $params

(Get-MgBetaDirectorySetting -DirectorySettingId $settingsObjectID).Values
```
