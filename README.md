bash-onedrive-upload
====================

Upload files to [Microsoft OneDrive](https://onedrive.live.com) via linux command line using the [REST API](http://onedrive.github.io/).

Use `git clone --recursive` to checkout the repository including all required submodules.

Prerequisites
-------------

- `curl` is used for accessing the API.
- `grep` is used for filtering the parsed json output.
- `cut` is used for value extraction from the filtered json output.
- `xargs` is used for multi-threaded file uploads.
- `dd` is used for chunked file uploads.
- `stat` is used to determine the filesize.

Getting started (OneDrive Personal)
---------------

Before you can use this tool, create an application in the [Microsoft Application Registration Portal](https://https://apps.dev.microsoft.com/#/appList).

Please add a new `Converged application`. The old `Live SDK applications` are **no longer supported** by this script.

Configure the newly created application as follows:

- Create a password by clicking `Generate New Password`.
- Create a new platform by clicking `Add Platform`. Choose `Web` and enter a `Redirect URI`. You can use `http://localhost`. This URI will only be used for the authorization process.
- Make sure the checkbox for `Live SDK support` is checked.

By now you should have acquired three values

    Application ID: 5e4504e0-e176-4a4b-8c6e-4313812b72d2
    Applications password: 7JLy4n9HB4Ym6eUCQtyxh0K
    Redirect URI: http://localhost

Please insert these values in the matching variables in `onedrive.cfg`:

    export api_client_id="5e4504e0-e176-4a4b-8c6e-4313812b72d2"
    export api_client_secret="7JLy4n9HB4Ym6eUCQtyxh0K"
    export api_reply_url="http://localhost"

After the initial configuration you must authorize the app to use your OneDrive account. Run

    $ ./onedrive-authorize

and follow the steps. You will need a web browser.

After the authorization process has successfully completed you can upload files.

Getting started (OneDrive for Business)
---------------

Before you can use this tool, create an application in the [Microsoft Azure Management Portal](https://manage.windowsazure.com) to generate your custom Client ID and Client secret.

    1. Navigate to Active directory
    2. Select your Directory
    3. Navigate to Applications tab
    4. Press Add
    5. Select "Add an application my organization developing"
    6. Choose Name and select "Web application and/or web API"
    7. In a "Sign-on URL" write https://login.microsoftonline.com/
    8. In an "APP ID URI" write https://onedrive.live.com/about/business/
    9. Submit form
    10. Now open your new application and select Configure tab
    11. Change "Reply URL" field to https://onedrive.live.com/about/business/
    12. Add an application key. Select 1 year key, 2 years key will not work. Press Save and copy the generated Secret
    13. Also copy your "Client ID"
    14. In a "Permissions to other applications" press "Add application"
    15. Select "Office 365 SharePoint Online" and "Windows Azure Active Directory"
    16. Grant "Sign in and read user profile" delegated permission to "Windows Azure Active Directory"
    17. Grant "Read user files" and "Read and write user files" delegated permissions to "Office 365 SharePoint Online"
    18. Save changes

Afterwards you should have three components of authorization process:

    Client ID: a66d1076-4c04-4a33-3bb2-2578c4891886
    Secret: x73LaIRCimtjiiw/cuHA000Ozwcf4nz1Ovcpi0xUCHI=
    Reply URL: https://onedrive.live.com/about/business/

Please insert these values in the matching variables in `onedriveb.cfg`:

    export api_client_id="a66d1076-4c04-4a33-3bb2-2578c4891886"
    export api_client_secret="x73LaIRCimtjiiw/cuHA000Ozwcf4nz1Ovcpi0xUCHI="
    export api_reply_url="https://onedrive.live.com/about/business/"

After the initial configuration you must authorize the app to use your OneDrive for Business account. Run

    $ ./onedriveb-authorize

and follow the steps. You will need a web browser.

After the authorization process has successfully completed you can upload files.
If you see error `AADSTS70002: Error validating credentials. AADSTS50012: Invalid client secret is provided`, try to add a new key and use new secret.

Usage (OneDrive Personal)
-----

To upload a single file simply type

    $ ./onedrive-upload file1

You can also upload multiple files, either by explicitly specifying each one

    $ ./onedrive-upload file1 file2

or just use wildcards (globbing)

    $ ./onedrive-upload file*.png

You can also specify a destination folder relative to the root folder configured in `onedrive.cfg`:

    $ ./onedrive-upload -f "relative/path" file1

This command will automatically determine all of the needed folder ids and recursively create all subfolders that do not yet exist.

If you need your file to be uploaded with a different filename, you can activate the renaming mode:

    $ ./onedrive-upload -r ./file1.txt renamed_file1.txt ./file2.txt renamed_file2.txt

Be aware that for each file you specify you must provide the remote filename as the subsequent parameter. This feature can lead to an unexpected behavior when combined with wildcards (globbing) because the pathname expansion is performed by bash before the execution of the script.

Usage (OneDrive for Business)
-----

To upload a single file simply type

    $ ./onedriveb-upload file1

You can also upload multiple files, either by explicitly specifying each one

    $ ./onedriveb-upload file1 file2

or just use wildcards (globbing)

    $ ./onedriveb-upload file*.png

You can also specify a destination folder relative to the root folder configured in `onedrive.cfg`:

    $ ./onedriveb-upload -f "relative/path" file1

This command will automatically determine all of the needed folder ids and recursively create all subfolders that do not yet exist.

If you need your file to be uploaded with a different filename, you can activate the renaming mode:

    $ ./onedriveb-upload -r ./file1.txt renamed_file1.txt ./file2.txt renamed_file2.txt

Be aware that for each file you specify you must provide the remote filename as the subsequent parameter. This feature can lead to an unexpected behavior when combined with wildcards (globbing) because the pathname expansion is performed by bash before the execution of the script.


Configuration
-------------

### Specify an alternate root folder for uploads

If you want to use a folder other than the root folder of your OneDrive as your upload root folder, you need to retrieve its unique id.

Open your the [OneDrive Web Interface](https://onedrive.live.com) and navigate to the folder, you want to configure as your new upload folder. The address bar should now show a URL like

    https://onedrive.live.com/?cid=123419ACDA5678AB&id=123419ACDA5678AB%2145321

The GET-parameter `id` is the unique folder id you need to place in your `onedrive.cfg`. You have to work with the decoded URL, e.g. `%21` decodes to `!`:

    export api_root_folder="items/123419ACDA5678AB!45321"

Unfortunately, you can't get item ID in this way for OneDrive for Business. So use another approach:

    Upload in debug mode any file to non-existent folder, which will be used as root folder.

    ./onedriveb-upload -d -f Backup testfile
    2016-08-26 09:33:56 Searching for 'Backup' in ''
    2016-08-26 09:33:56 Creating folder 'Backup' in ''
    2016-08-26 09:33:58 api_folder_id is now '01KB37ZVWEMG6F4SS6TVGZAM7ACZKOEERY'
    2016-08-26 09:33:58 Size of testfile is less than or equal to 104857600 bytes, will use simple upload
    2016-08-26 09:34:01 Uploading 'testfile' as 'testfile' into 01KB37ZVWEMG6F4SS6TVGZAM7ACZKOEERY

    Copy `api_folder_id` and place it in your `onedriveb.cfg`

    export api_root_folder="items/01KB37ZVWEMG6F4SS6TVGZAM7ACZKOEERY"

You can also specify any of the [special folders](https://dev.onedrive.com/items/special_folders.htm) provided by the API by simply using

    export api_root_folder="special/approot"
    export api_root_folder="special/documents"
    export api_root_folder="special/photos"
    ...

The default is to use

    export api_root_folder="root"

which identifies the root folder of your account.

Keep in mind that this will be the new root folder of your OneDrive as seen by this script. The script does *not* support escaping the root by using `../`.

### Less permissions for uploads

If you don't want the script to have access to all your folders and files you can request less permissions during the authorization process. Simply change the value

    export api_permissions="https://graph.microsoft.com/files.readwrite"

to

    export api_permissions="https://graph.microsoft.com/files.readwrite.appfolder"

and run the authorization process. You might need to remove the apps permissions prior to that ([Link](https://account.live.com/consent/Manage)) if you have already requested `onedrive.readwrite` permissions.

### Number of simultaneous uploads

If the script does not fully utilize your bandwidth, you can maybe speed things up a little bit by increasing the value of `max_upload_threads`.

When you start the upload of more than one file, the script will start up to `max_upload_threads` parallel uploads, but only one thread per file.

### Configure threshold for session based upload

For small files the script uses the [simple upload](https://dev.onedrive.com/items/upload_put.htm) of the api. For files that are larger than 100 MiB the script uses the [session based upload](https://dev.onedrive.com/items/upload_large_files.htm).

If you want to use the session based upload for files smaller than 100 MiB you can change the value of `max_simple_upload_size` to any positive value smaller than 104857600:

    export max_simple_upload_size=52428800

You can also change the value of `max_chunk_size` to any positive value smaller than 62914560, if you want to use smaller or larger chunks than the default:

    export max_chunk_size=62914560

Contributors
------------

- [rootik](https://github.com/rootik) (OneDrive for Business support)
- [huming2207](https://github.com/huming2207) (OSX support)
- [laufi](https://github.com/laufi)

Thank you all for your valuable contributions.
