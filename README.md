# Youtube Uploader

Scripted uploads to youtube.

- upload video files from local disk or from the web.
- ratelimit upload bandwidth

## Download

Grab a [precompiled binary](https://github.com/porjo/youtubeuploader/releases) for Linux, Mac or Windows or build it yourself.

## Setup

### Youtube API

Talking to the Youtube API requires oauth2 authentication. As such, you must:

1. Create an account on the [Google Developers Console](https://console.developers.google.com)
1. Create a new project for this app
1. Enable the Youtube API (APIs & Services -> Enable APIs and Services -> Click 'ENABLE APIS AND SERVICES' top right). Select 'YouTube Data API v3'
1. Create OAuth consent screen (APIs & Services -> OAuth Consent Screen)
   - Add a test user. This can be any Google User account but it should correspond with the Youtube account where videos will be uploaded.
1. Create Credentials (APIs & Services -> Credentials -> click 'CREATE CREDENTIALS'), select 'OAuth client ID', select 'Web application'
   - Add an 'Authorized redirect URI' of 'http://localhost:8080/oauth2callback'
1. Download the client secrets JSON file (click download icon next to newly created client ID) and save it as file `client_secrets.json` in the same directory as the utility e.g.

```
{
  "web": {
    "client_id": "xxxxxxxxxxxxxxxxxxxxxxxx.apps.googleusercontent.com",
    "project_id": "youtubeuploader-yyyyy",
    "auth_uri": "https://accounts.google.com/o/oauth2/auth",
    "token_uri": "https://oauth2.googleapis.com/token",
    "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
    "client_secret": "xxxxxxxxxxxxxxxxxxxx",
    "redirect_uris": [
      "http://localhost:8080/oauth2callback"
    ]
  }
}
```

## Usage

At a minimum, just specify a filename:

```
./youtubeuploader -filename blob.mp4
```

If it is the first time you've run the utility, a browser window should popup and prompt you to provide Youtube credentials. A token will be created and stored in `request.token` file in the local directory for subsequent use. To run the utility on a headless-server, generate the token file locally first, then simply copy the token file along with `youtubeuploader` and `client_secrets.json` to the remote host.

Full list of options:
```
  -cache string
    	token cache file (default "request.token")
  -caption string
    	caption filename. Can be a URL
  -categoryId string
    	video category Id
  -chunksize int
    	size (in bytes) of each upload chunk. A zero value will cause all data to be uploaded in a single request (default 16777216)
  -debug
    	turn on verbose log output
  -description string
    	video description (default "uploaded by youtubeuploader")
  -filename string
    	video filename. Can be a URL. Read from stdin with '-'
  -language string
    	video language (default "en")
  -limitBetween string
    	only rate limit between these times e.g. 10:00-14:00 (local time zone)
  -metaJSON string
    	JSON file containing title,description,tags etc (optional)
  -metaJSONout string
    	filename to write uploaded video metadata into (optional)
  -notify
    	notify channel subscribers of new video. Specify '-notify=false' to disable. (default true)
  -oAuthPort int
    	TCP port to listen on when requesting an oAuth token (default 8080)
  -privacy string
    	video privacy status (default "private")
  -quiet
    	suppress progress indicator
  -ratelimit int
    	rate limit upload in Kbps. No limit by default
  -secrets string
    	Client Secrets configuration (default "client_secrets.json")
  -tags string
    	comma separated list of video tags
  -thumbnail string
    	thumbnail filename. Can be a URL
  -title string
    	video title
  -version
    	show version
```
*NOTE:* When specifying a URL as the filename, the data will be streamed through the localhost (download from remote host, then upload to Youtube)

If `-quiet` is specified, no upload progress will be displayed. Current progress can be output by sending signal `USR1` to the process e.g. `kill -USR1 <pid>` (Linux/Unix only).

### Metadata

Video title, description etc can specified via the command line flags or via a JSON file using the `-metaJSON` flag. An example JSON file would be:

```json
{
  "title": "my test title",
  "description": "my test description",
  "tags": ["test tag1", "test tag2"],
  "privacyStatus": "private",
  "madeForKids": false,
  "embeddable": true,
  "license": "creativeCommon",
  "publicStatsViewable": true,
  "publishAt": "2017-06-01T12:05:00+02:00",
  "categoryId": "10",
  "recordingdate": "2017-05-21",
  "playlistIds":  ["xxxxxxxxxxxxxxxxxx", "yyyyyyyyyyyyyyyyyy"],
  "playlistTitles":  ["my test playlist"],
  "language":  "fr"
}
```
- all fields are optional. Command line flags will be used by default (where available)
- use `\n` in the description to insert newlines
- times can be provided in one of two formats: `yyyy-mm-dd` (UTC) or `yyyy-mm-ddThh:mm:ss+zz:zz`

## Alternative Oauth setup for headless clients

NOTE: this option has been removed as it is no longer supported by Google. See [issue #116](https://github.com/porjo/youtubeuploader/issues/116) for more information.

## Releases

Releases are built with [Goreleaser](https://github.com/goreleaser/goreleaser):
```
# Signed tag
git tag -s <tag>
git push --tags
GITHUB_TOKEN=xxxxyyy goreleaser release
```

## Credit

Based on [Go Youtube API Sample code](https://github.com/youtube/api-samples/tree/master/go)

Thanks to [github.com/tokland/youtube-upload](https://github.com/tokland/youtube-upload) for insight into how to update playlists.
