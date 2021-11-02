# Release Checklist

## Windows
The following steps should be taken to release a new version of the software for Windows.

1. Increment versions in the following projects:
	1. StatTag
	2. Core
	3. Jupyter
	4. R
	5. SAS
	6. Stata
2. Update Setup-x86
	1. Click the 'Setup-x86' project in the Solution Explorer to view its properties
		1. Update the Version
			1. Note that Setup projects only allow X.Y.Z versions, and StatTag uses X.Y.Z.A. To accommodate this, we use the "Z" portion of the version to include "Z.A".  For example, version 6.0.3.2 would have an installer version of 6.0.302.  Version 6.1.10.55 would have an installer version 6.1.1055, etc.
		2. Visual Studio should prompt you to regenerate the Product Code. Confirm that you want to regenerate it.  Otherwise, go to the Product Code in the Properties and regenerate it.
	2. Change the build configuration to "Release"
	3. Change the platform to "x86"
	4. Clean and then rebuild the entire solution
	5. Copy the setup file to the shared releases folder
3. Click the 'Setup-x64' project in the Solution Explorer to view its properties
	1. Repeat steps 2.1 and 2.2
	2. Change the platform to "Any CPU"
	3. Perform steps 2.4 and 2.5


### Notes
The x64 and x86 Upgrade Code (which should never be changed) is: {AEC2B59D-045C-4D6D-A4B4-1C87125EA095}

We have purposely kept the same upgrade code for all platforms.

### Additional information
Starting with release v6.0.3 we moved away from the InstallShield Limited Edition installer and moved to Microsoft's Setup Project installer.  The Setup Project closely mimics what we had available in InstallShieled LE, with one exception that it generates an MSI instead of an EXE, and some minor installer UI differences.

The Setup project should be configured to replace older InstallShield installs, so there shouldn't be continuity issues moving from one platform to the next.  This is because of the Upgrade Code that we have fixed.


## macOS
Update the version for the application in Xcode following our [versioning conventions](https://github.com/StatTag/stattag-documentation/blob/master/Versioning.md).  Perform `Product -> Clean`, and `Product -> Build` to ensure the build is refreshed.

Generate the application via `Product -> Archive`.  When the Archive window appears (assuming it completes without error), ensure your build is selected in the list and click `Export...`.  Select `Export a Developer ID-signed Application`, select the `Northwestern University` Development Team, click the `Export` button on the Summarys creen, and define a location for the application to be exported to.

##### Checking Framework References

From the command line, navigate to the directory where the StatTag.app exists.  We need to check that our weak references aren't bound to a specific version of R.  You can check this by running:

`otool -L StatTag.app/Contents/Frameworks/RCocoa.framework/RCocoa | grep R.framework`

Results like this mean we have a problem, since a specific R version (3.6) is referenced in the R.framework path.

```
/Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libR.dylib (compatibility version 3.6.0, current version 3.6.3)
```

Results like this are okay, since it's already pointing to the `Current` path for R.framework:

```
/Library/Frameworks/R.framework/Versions/Current/Resources/lib/libR.dylib (compatibility version 3.6.0, current version 3.6.3)
```

If you do encounter a problem with RCocoa pointing to a specific version of R.framework, the following commands can fix that.  This also re-signs the application, which is important.  The `install_name_tool` invalidates our previous signature otherwise.

> **NOTE**: You will need to change the commands for the specific version of the R.framework that you found from `otool`.

```
install_name_tool -change /Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libR.dylib /Library/Frameworks/R.framework/Versions/Current/Resources/lib/libR.dylib StatTag.app/Contents/Frameworks/RCocoa.framework/RCocoa

install_name_tool -change /Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libR.dylib /Library/Frameworks/R.framework/Versions/Current/Resources/lib/libR.dylib StatTag.app/Contents/Frameworks/RCocoa.framework/Versions/A/RCocoa

install_name_tool -change /Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libR.dylib /Library/Frameworks/R.framework/Versions/Current/Resources/lib/libR.dylib StatTag.app/Contents/Frameworks/RCocoa.framework/Versions/Current/RCocoa

install_name_tool -change /Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libR.dylib /Library/Frameworks/R.framework/Versions/Current/Resources/lib/libR.dylib StatTag.app/Contents/Frameworks/StatTagFramework.framework/StatTagFramework

install_name_tool -change /Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libR.dylib /Library/Frameworks/R.framework/Versions/Current/Resources/lib/libR.dylib StatTag.app/Contents/Frameworks/StatTagFramework.framework/Versions/A/StatTagFramework

install_name_tool -change /Library/Frameworks/R.framework/Versions/3.6/Resources/lib/libR.dylib /Library/Frameworks/R.framework/Versions/Current/Resources/lib/libR.dylib StatTag.app/Contents/Frameworks/StatTagFramework.framework/Versions/Current/StatTagFramework


codesign --deep --force --verify --verbose --sign "Developer ID Application: Northwestern University" StatTag.app

codesign -dv --verbose=4 StatTag.app
```

##### Packaging
Find a pre-existing or template DMG which contains the StatTag icon.  If there is an existing version of StatTag.app within the DMG, move it to the Trash.  If you are also updating the User's Guide, move that to the Trash as well.  **Empty the Trash**.

> If the DMG runs out of space, the following command will let you increase its capacity.  Be sure to change 17M to whatever maximum size you need:
	
>	`hdiutil resize -size 20.5M StatTag-TEMPLATE.dmg`

Copy the new StatTag.app into the DMG, and position it appropriately in the view.  Do the same with the User's Guide if it is also being updated.  Make sure that the view is exactly how you want it to appear to users (e.g., hiding the Favorites bar, appropriately sized).

Run Disk Utility.  You should see your StatTag DMG mounted.  Select it in the list, and from the menu select File -> New Image -> Image from "StatTag".  Ensure the format is "read-only".

Eject the StatTag image, and test that the newly created DMG mounts and automatically opens the view for the user.

Rename the DMG based on the version, following the naming conventions.  You are now ready to upload this to the download servers.

### Alternate Signing Instructions ###

If for any reason you are unable to export as a Developer ID-signed application, but you do have a valid developer ID, the following alternate steps should work.  This should not be your main method, but instructions are included here just in case.

As before, begin an `Export...`.  Select `Export as a macOS App` and define a location for the application to be exported to.

Sign the generated application with our developer key.  Note that this assumes you're already in the directory where you have exported StatTag.app.
```
codesign --deep --force --verify --verbose --sign "Developer ID Application: Northwestern University" StatTag.app
```

Optionally, you can run the following to confirm that the app has been signed:
```
codesign -dv --verbose=4 StatTag.app
```
