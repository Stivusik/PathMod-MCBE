> For LiteLDev, OpenMCBE and **any** of their subcompanies/suborganizations is not allowed to use this product for their purposes in any way.

# Replace strings
Replace all strings from:
```
invoke-virtual {p0}, Lcom/mojang/minecraftpe/MainActivity;->getDataDir()Ljava/io/File;
```
```
const/4 v0, 0x0

invoke-virtual {p0, v0}, Lcom/mojang/minecraftpe/MainActivity;->getExternalFilesDir(Ljava/lang/String;)Ljava/io/File;
```

To:
```
invoke-static {}, Landroid/os/Environment;->getExternalStorageDirectory()Ljava/io/File;
```
In MainActivity.smali

# Add permission request:
Paste these methods in MainActivity.smali:
```
.method public IfStoragePermissionWasDenied(Landroid/content/Context;)V
    .registers 4
    .param p1, "context"  # Landroid/content/Context;

    .prologue
    sget v0, Landroid/os/Build$VERSION;->SDK_INT:I

    const/16 v1, 0x1e

    if-lt v0, v1, :cond_1f

    invoke-static {}, Landroid/os/Environment;->isExternalStorageManager()Z

    move-result v0

    if-eqz v0, :cond_d

    return-void

    :cond_d
    const-string v0, "Storage permission is required"

    const/4 v1, 0x1

    invoke-static {p1, v0, v1}, Landroid/widget/Toast;->makeText(Landroid/content/Context;Ljava/lang/CharSequence;I)Landroid/widget/Toast;

    move-result-object v1

    invoke-virtual {v1}, Landroid/widget/Toast;->show()V

    invoke-static {}, Landroid/os/Process;->myPid()I

    move-result v1

    invoke-static {v1}, Landroid/os/Process;->killProcess(I)V

    return-void

    :cond_1f
    return-void
.end method

.method public RequestPermission()V
    .registers 9
    .annotation build Landroid/annotation/SuppressLint;
        value = {
            "NewApi"
        }
    .end annotation

    .prologue
    const/4 v7, 0x0

    const/4 v6, 0x1

    const/4 v5, 0x0

    sget v3, Landroid/os/Build$VERSION;->SDK_INT:I

    const/16 v4, 0x1e

    if-lt v3, v4, :cond_26

    invoke-static {}, Landroid/os/Environment;->isExternalStorageManager()Z

    move-result v3

    if-nez v3, :cond_26

    new-instance v3, Landroid/content/Intent;

    sget-object v4, Landroid/provider/Settings;->ACTION_MANAGE_APP_ALL_FILES_ACCESS_PERMISSION:Ljava/lang/String;

    invoke-direct {v3, v4}, Landroid/content/Intent;-><init>(Ljava/lang/String;)V

    const-string v4, "package"

    invoke-virtual {p0}, Landroid/app/Activity;->getPackageName()Ljava/lang/String;

    move-result-object v5

    invoke-static {v4, v5, v7}, Landroid/net/Uri;->fromParts(Ljava/lang/String;Ljava/lang/String;Ljava/lang/String;)Landroid/net/Uri;

    move-result-object v4

    invoke-virtual {v3, v4}, Landroid/content/Intent;->setData(Landroid/net/Uri;)Landroid/content/Intent;

    invoke-virtual {p0, v3}, Landroid/app/Activity;->startActivity(Landroid/content/Intent;)V

    :cond_26
    return-void
.end method
```

# After adding RequestPermission()V and IfStoragePermissionWasDenied(Landroid/content/Context;)V, add this string to the start of OnCreate:
```
invoke-virtual {p0}, Lcom/mojang/minecraftpe/MainActivity;->RequestPermission()V

invoke-virtual {p0, p0}, Lcom/mojang/minecraftpe/MainActivity;->IfStoragePermissionWasDenied(Landroid/content/Context;)V
```
It should look like this:
<img width="1080" height="197" alt="Screenshot_20251214_114646_MT Manager" src="https://github.com/user-attachments/assets/02225a50-141b-41c7-bd32-881249a2b2ba" />

# Add permissions in AndroidManifest.xml
Paste this anywhere:
```
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
```
```
<uses-permission android:name="android.permission.MANAGE_EXTERNAL_STORAGE" />
```
Remove ```android:maxSdkVersion="32"```:
<img width="1080" height="100" alt="Screenshot_20251214_115028_MT Manager" src="https://github.com/user-attachments/assets/a4309a8a-9b25-4c3b-beb3-47f77e170d24" />


# Scripts
If you are lazy, you can decompile mc apk using ApkTool M and run the script.
The shell script was tested with MPatcher and MT Manager, and the python script was tested with Pydroid 3.
