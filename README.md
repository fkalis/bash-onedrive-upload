bash-onedrive-upload
====================

Upload files to [Microsoft OneDrive](https://onedrive.live.com) via linux command line using the [REST API](http://msdn.microsoft.com/library/dn659752.aspx).

Prerequisites
-------------

This script uses `curl` for accessing the API, and `grep` and `cut` for value extraction.

Getting started
---------------

Before you can use this tool, create an application in the [Microsoft account Developer Center](https://account.live.com/developers/applications) to generate your custom Client ID and Client secret. Notice, that you need to switch the API Setting "Mobile or desktop client app" to "yes".

You should now be in possession of credentials in the form

    Client ID: 00000000A2B3C495
    Client secret: 88qC3kX2Cbd0tXV2sBnqYbS321abcDEF

Now insert these values in the matching variables in `onedrive.cfg`.

Usage
-----

After the initial configuration you must authorize the app to use your OneDrive account. Run

    $ ./onedrive-authorize

and follow the steps. You will need a web browser.

After the authorization process has successfully completed you can now upload files (up to 100MB each due to API limitations):
    
    $ ./onedrive-upload file1
    $ ./onedrive-upload file1 file2
    $ ./onedrive-upload file*.png

You can also specify a destination folder relative to the folder configured in `onedrive.cfg`:

    $ ./onedrive-upload -f "relative/path" file1

This command will automatically determine all of the needed folder ids and recursively create all subfolders that do not yet exist.

Configuration
-------------

### Specify an alternate upload folder

If you want to use a folder other than the root folder of your OneDrive, you need to find out its unique folder id.

Open your the [OneDrive Web Interface](https://onedrive.live.com) and navigate to the folder, you want to configure as your new upload folder. The address bar should now show a URL like

    https://onedrive.live.com/?cid=123419ACDA5678AB&id=123419ACDA5678AB%2145321

The folder id can be constructed from the parameters `cid` and `id`:

    folder.${cid}.${id}

The folder id for the example URL reads as follows (you have to work with the decoded URL, e.g. `%21` decodes to `!`):

    folder.123419ACDA5678AB.123419ACDA5678AB!45321

### Number of simultaneous uploads

If the script does not fully utilize your bandwidth, you can maybe speed things up a little bit by increasing the value of `max_upload_threads`.

When you now start the upload of more than one file, the script will start up to `max_upload_threads` parallel uploads, but only one thread per file.
