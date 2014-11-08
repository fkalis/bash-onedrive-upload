bash-onedrive-upload
====================

Upload files to [Microsoft OneDrive](https://onedrive.live.com) via linux command line using the [REST API](http://msdn.microsoft.com/library/dn659752.aspx).

Currently this script can only upload to one specific folder (configured in `onedrive.cfg`).

Prerequisites
-------------

This script uses `curl` for accessing the API and `jq` for parsing the json data. `jq` can be found [here](http://stedolan.github.io/jq/).

Getting started
---------------

Before you can use this tool, create an application in the [Microsoft account Developer Center](https://account.live.com/developers/applications) to generate your custom Client ID and Client secret. Notice, that you need to switch the API Setting "Mobile or desktop client app" to "yes".

You should now be in possesion of credentials in the form

    Client ID: 00000000A2B3C495
    Client secret: 88qC3kX2Cbd0tXV2sBnqYbS321abcDEF

Now insert these values in the matching variables in `onedrive.cfg`.

Usage
-----

After the initial configuration you must authorize the app to user your OneDrive account. Run

    $ ./onedrive-authorize

and follow the steps. You will need a web browser.

After the authorization process has successfully completed you can now upload files (up to 100MB each due to API limitations):
    
    $ ./onedrive-upload file1
    $ ./onedrive-upload file1 file2
    $ ./onedrive-upload file*.png
