# Release Checklist

## Windows
The following steps should be taken to release a new version of the software for Windows.

1. Increment versions in the following projects:
	1. StatTag
	2. Core
	3. Stata
	4. SAS
	5. R
2. Reload the OfficeAddInSetup-x86 project
	1. Under Organize Your Setup > General Information
		1. Update the Product Version
		2. Regenerate the Product Code
		3. Change the upgrade path to include the previous version
	2. Change the build configuration to "Release"
	3. Change the platform to "x86"
	4. Clean and then rebuild the entire solution
	5. Copy the setup file to the shared releases folder
3. Unload the OfficeAddInSetup-x86 project and reload the OfficeAddInSetup-x64 project
	1. Repeat steps 2.1 and 2.2
	2. Change the platform to "Any CPU"
	3. Perform steps 2.4 and 2.5


### Notes
The x64 and x86 Upgrade Code (which should never be changed) is: {AEC2B59D-045C-4D6D-A4B4-1C87125EA095}

We have purposely kept the same upgrade code for all platforms.

### Additional information
Setting up InstallShield to allow upgrades was found at: [http://stackoverflow.com/a/13874942/5670646](http://stackoverflow.com/a/13874942/5670646)

Original setup instructions were:


> "In your InstallShield setup project, you should do the following:

> * select branch: Organize your setup -> Upgrade Paths
* add new upgrade path and than press the cancel button
* the default properties of the new upgrade path should not be changed if you do not plan to change the Product version from the following branch: Organize your setup -> General Information. If you plan to change the Product version, than you should play with the following upgrade path properties: Min Version/_Include Min Version_, Max Version/_Include Max Version_.
* every time you need to create a new setup, change the Product code from the following branch: Organize your setup -> General Information.
* please be aware that the Upgrade Code should NEVER be changed."

## macOS
Update the version for the application in Xcode following our [versioning conventions](https://github.com/StatTag/stattag-documentation/blob/master/Versioning.md).  Perform `Product -> Clean`, and `Product -> Build` to ensure the build is refreshed.

Generate the application via `Product -> Archive`.  When the Archive window appears (assuming it completes without error), ensure your build is selected in the list and click `Export...`.  Select `Export a Developer ID-signed Application`, select the `Northwestern University` Development Team, click the `Export` button on the Summarys creen, and define a location for the application to be exported to.

Find a pre-existing or template DMG which contains the StatTag icon.  If there is an existing version of StatTag.app within the DMG, move it to the Trash.  If you are also updating the User's Guide, move that to the Trash as well.  **Empty the Trash**.

If the DMG runs out of space, the following command will let you increase its capacity.  Be sure to change 17M to whatever maximum size you need:
	
	hdiutil resize -size 17M StatTag-TEMPLATE.dmg

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
