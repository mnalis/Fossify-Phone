# Fossify Phone

## Summary
This is unofficial fork of Fossify phone from https://www.github.com/FossifyOrg

as a temporary fix to have notifications  by implementing
never-fully-finished https://github.com/FossifyOrg/Phone/pull/253 until 
https://github.com/FossifyOrg/Phone/issues/83 is more properly fixed.

## Build instructions

This is already done by this repo, but if you want to reproduce it yourself,
you need to: 

1. fork fossify phone from https://github.com/FossifyOrg/Phone

2. checkout 1.6.0 version (last one that the following PR applies cleanly)

3. apply PR from https://github.com/FossifyOrg/Phone/pull/253
   to fix notifications on missed calls (otherwise you'll not see missed calls!)
   e.g. `curl -s https://github.com/FossifyOrg/Phone/pull/253.diff | patch -p1`

4. setup GitHub to build the app:

4.1. install prerequisites 
     on Debian Bookworm: `apt install openjdk-17-jre-headless adb` # to get keytool and adb

4.2. generate signing key
     ```
      keytool -genkeypair \
      -v \
      -keystore release.keystore \
      -alias release \
      -keyalg RSA \
      -keysize 4096 \
      -validity 10000
     ```

4.3. On GitHub fork of FossifyPhone settings, under Security, click "Secrets and Variables"
     and then "Actions" and then under "Secrets" tab setup "Repository
     secrets" by clicking 4 times on "New repository secret"

     - RELEASE_KEYSTORE_BASE64: (output from "base64 -w 0 release.keystore")
     - SIGNING_KEY_ALIAS: release (because of "-alias relase" in keytool)
     - SIGNING_KEY_PASSWORD: (password from keytool - make sure there is no trailing whitespace or newline!)
     - SIGNING_STORE_PASSWORD: (same as above)

4.4. create .github/workflows/build-release-apk.yml in your fork to build signed release
     see https://github.com/mnalis/Fossify-Phone/blob/master-mn/.github/workflows/build-release-apk.yml
     for example

4.5 in your GitHub fork of FossifyPhone, click "Actions", "Build signed release apk", "Run workflow",
    choose your branch with all the changes, and click green "Run workflow" button

4.6 wait ~5 minutes so it produces `foss-release-signed.zip`

4.7. download the `.zip` to your phone, extrack `.apk` from it and install it



## Install and configure the app

5. setup the app on the phone:

5.1. long click on the Fossify Phone app icon, select `(i)` and add all permission that it wants to the app.
     also see three-dots menu and "Special access" and find all that mention Fossify Phone and allow it (e.g. "Appear on top")

5.2. in android "Choose defualt apps", you must make the Fossify Phone app the default for both
     "Called ID & spam app" and "Phone app". on SamsungGalaxy S23+ with OneUI 7.0 and Android 15, the latter would refuse.

     workaround which worked for me is to have "USB debugging" in "Developer options" (search the net on how to enable
     that hidden menu), and then connect phone with computer, and issue following commands:

     ```
     adb shell cmd role add-role-holder android.app.role.DIALER org.fossify.phone
     adb shell cmd role get-role-holders android.app.role.DIALER # verify if it worked - should return "org.fossify.phone"
     adb shell pm grant org.fossify.phone android.permission.READ_CALL_LOG
     adb shell pm grant org.fossify.phone android.permission.WRITE_CALL_LOG
     adb shell pm grant org.fossify.phone android.permission.READ_CONTACTS
     adb shell pm grant org.fossify.phone android.permission.READ_PHONE_STATE
     adb shell pm grant org.fossify.phone android.permission.ANSWER_PHONE_CALLS
     #adb shell dumpsys role # if need to debug system data only
     adb shell settings get secure dialer_default_application # returned "null" for me, which is apprently OK
     ```

5.3. (optional?) uninstall Samsung Dialer:

     ```
     adb shell pm uninstall --user 10 com.samsung.android.dialer      # might fail on your system if you didn't use work profile
     adb shell pm uninstall --user 0 com.samsung.android.dialer       # (alternative is "adb shell pm disable-user --user 0 com.samsung.android.dialer")
     ```

5.4. reboot

5.5. test receiving calls, making calls, and seeing missed calles
