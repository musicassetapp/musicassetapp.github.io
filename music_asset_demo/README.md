# Music Asset Demo

## How to use?

### Getting Authorized
To host this web app, you have to be a YouTube content owner, create a Google API project, enable the correct Google APIs, and get a client ID.
* Visit the [Google API Console](https://console.developers.google.com/flows/enableapi?apiid=picker&credential=client_key).
    * Make sure you're logged in as a content owner.
* Register your application for Google Picker API and go to credentials.
* Under API Manager, navigate to Library.
* Search and enable the:
    1. Google+ API
    2. YouTube Content ID API (you will need to be logged in as a content owner)
        * Click create credentials and get your API key.
* Navigate back to the Credential under API Manager if you're not there already.
* Use the OAuth consent screen tab to set a product name.
* Under the credentials tab, click create credentials and hit "OAuth Client ID."
* Choose web application as your application type, enter your host website's URL under "Authorized JavaScript origins," and create.
* Copy and paste the client ID into the string variable "CLIENT_ID" in authorization.html.

### Making a Search
The search will require you to be logged in to an account linked to one or more content owners. To search, choose between ISRC and ISWC search, enter the appropriate code, and submit. You may also use the 'Advanced Search' option to customize what metadata fields the search results should display. Read on for details about each search...

### ISRC Search
ISRC search finds sound recording assets with ISRCs that match the query and displays every version of each of those
assets. The asset versions are grouped by asset ID, and the versions which belong to either you or the content owner you are searching on behalf of are highlighted in light red. ISRC search also finds composition assets linked to these sound recording assets and displays the reconciled metadata of those assets. The ISWC codes of these assets are hyperlinked to ISWC search.

### ISWC Search
ISWC search finds sound recording assets that are linked to compositions with the provided ISWC and displays the reconciled metadata of these assets. The ISRC codes of these assets are hyperlinked to ISRC search. The API call to get the asset metadata is limited to a certain number of asset IDs per call (the limit is currently 50). If there are more asset IDs than this limit found, you can click load all, load more, or simply scroll to load more data. The API call to get the asset IDs is also limited (the limit is currently 500), so these loading features will stop at this limit.

## API Calls

### Authorization
These calls occur on log in.

#### Auth
The first call is to the Google Authentication API. This call requires your client ID, which you can find and set following the instructions under **Getting Authorized**. The response provides an access token that is required for all future API calls.

#### User ID: https://developers.google.com/+/web/api/rest/latest/people/get
This call is to the Google Plus API's **People: get**. It takes in the access token and its response contains your user ID.

#### Content ID: https://developers.google.com/youtube/partner/docs/v1/contentOwners/list
This call is to the YouTube Content ID API's **ContentOwners: list**. It takes in the access token, "fetchMine=true", and "onBehalfOfContentOwner=*Your Content Owner ID*". Its response contains all content owner IDs linked to your account.

### ISRC Search

#### AssetSearch: https://developers.google.com/youtube/partner/docs/v1/assetSearch/list
The first call is to the YouTube Content ID API's **AssetSearch: list**. We use it to find the sound recording assets linked to the provided ISRC. It takes in the access token, "isrcs=*the provided ISRC*", "ownershipRestriction=none", "type=sound_recording", and "onBehalfOfContentOwner=*Your Content Owner ID*". Its response contains an array of asset objects.

#### MetadataHistory: https://developers.google.com/youtube/partner/docs/v1/metadataHistory/list
This call is to the YouTube Content ID API'S **MetadataHistory: list**. We use this call to find the different versions of the sound recording assets we found using AssetSearch: list. This call takes in the access token, a string of asset IDs delimited by commas, and "onBehalfOfContentOwner=*Your Content Owner ID*". Its response contains an array of asset version objects. We display the information contained in these objects in the "Sound Recording" section.

#### AssetRelationships: https://developers.google.com/youtube/partner/docs/v1/assetRelationships/list
This call is to the YouTube Content ID API's **AssetRelationships: list**. We use this call to find the composition assets linked to the sound recording assets we found using AssetSearch: list. This call takes in the access token, an asset ID, and "onBehalfOfContentOwner=*Your Content Owner ID*" and its response contains an array of relationship objects. The relevant fields of this object are parentAssetID, which is the sound recording asset ID, and childAssetID, which is the asset ID of the composition linked to the sound recording. We make this call for each of the assets found using AssetSearch.

#### ContentOwners: https://developers.google.com/youtube/partner/docs/v1/contentOwners/list
This call is to the YouTube Content ID API's **ContentOwners: list**. We use this call to find content owner information on the content owner IDs found in the MetadataHistory: list response. This call takes in the access token, a string of owner IDs delimited by commas, and "onBehalfOfContentOwner=*Your Content Owner ID*". Its response contains an array of contentOwner objects which we use to link the content owner ID to the content owner display name.

#### Assets: https://developers.google.com/youtube/partner/docs/v1/assets/list
This call is to the YouTube Content ID API's **Assets: list**. We use this call two times during ISRC search. The first is to find the reconciled metadata of the compositions we found using AssetRelationships: list. It takes in the access token, a string of asset IDs delimited by commas (from AssetRelationships), "fetchMetadata=effective", and "onBehalfOfContentOwner=*Your Content Owner ID*". Its response contains an array of composition asset objects. We display the information contained in these objects in the "Compositions" section. The second call is to find the ownership territories of any sound recording versions owned by the currently authorized content owner found in the MetadataHistory: list response. The call takes in the access token, a string of asset IDs that the content owner owns versions of, "fetchMetadata=effective", "fetchOwnership=mine", "fetchOwnershipConflicts=true", and "onBehalfOfContentOwner=*Your Content Owner ID*". In the response, we find the territories where the user owns the asset and any territory conflicts, then display the information.

### ISWC Search

#### AssetSearch: https://developers.google.com/youtube/partner/docs/v1/assetSearch/list
The first call is to the YouTube Content ID API's **AssetSearch: list**. We use it to find the sound recording assets linked to the provided ISWC. It takes in the access token, "ownershipRestriction=none", "type=sound_recording", "metadataSearchFields=iswc:*The Provided ISWC*", and "onBehalfOfContentOwner=*Your Content Owner ID*". Its response contains an array of sound recording asset objects.

#### Assets: https://developers.google.com/youtube/partner/docs/v1/assets/list
This call is to the YouTube Content ID API's **Assets: list**. We use this call to find the reconciled metadata of the sound recordings we found using AssetSearch: list. It takes in the access token, a string of asset IDs delimited by commas (from AssetSearch), "fetchMetadata=effective", and "onBehalfOfContentOwner=*Your Content Owner ID*". Its response contains an array of sound recording asset objects. We display the information contained in these objects in the "Sound Recording" section.
