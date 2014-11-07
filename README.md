bash-onedrive-upload
====================

Upload files to onedrive via linux command line

Prerequisites
-------------

This script uses `curl` for accessing the API and `jq` for parsing the json data. `jq` can be found [here](http://stedolan.github.io/jq/).

Getting started
---------------

Before you can use this tool, create an application in the [Microsoft account Developer Center](https://account.live.com/developers/applications) to generate your custom Client ID and Client secret. Notice, that you need to switch the API Setting "Mobile or desktop client app" to "yes".

You should now be in possesion of credentials in the form
    Client ID: 00000000A2B3C495
    Client secret: 88qC3kX2Cbd0tXV2sBnqYbS321abcDEF

TODO: Description how to generate initial refresh_token.

Usage
-----

After configuration you can now upload files (up to 100MB):
    
    $ ./onedrive-upload file1
    $ ./onedrive-upload file1 file2
    $ ./onedrive-upload file*.png

