# ovos-media-plugin-spotify

spotify plugin for [ovos-audio](https://github.com/OpenVoiceOS/ovos-audio) and [ovos-media](https://github.com/OpenVoiceOS/ovos-media)

Allows OVOS to initiate playback on Spotify 

> NOTE: [the companion skill](https://github.com/OpenVoiceOS/skill-ovos-spotify) is needed to integrate with voice search

## Install

`pip install ovos-media-plugin-spotify`



## Play Spotify on another device

To play Spotify on connected speakers or other devices remotely via OVOS voice commands, follow these steps:

### Spotify developer account
1. Log in at [developer.spotify.com](https://developer.spotify.com) and set up a developer account, create an "Application".
2. Add "https://localhost:8888" as the redirect URL, and choose 'add'.
3. Accept the terms and choose save.
4. Go to settings.
5. Copy the ClientID


### OAuth
Currently Oauth needs to be performed manually.
Make sure the ovos-media-plugin-spotify is installed. 

1. Run `ovos-spotify-oauth` on the command line and follow the instructions

```
$ ovos-spotify-oauth
This script creates the token information needed for running spotify
        with a set of personal developer credentials.

        It requires the user to go to developer.spotify.com and set up a
        developer account, create an "Application" and make sure to whitelist
        "https://localhost:8888".

        After you have done that enter the information when prompted and follow
        the instructions given.
        
YOUR CLIENT ID: xxxxx
YOUR CLIENT SECRET: xxxxx
Go to the following URL: https://accounts.spotify.com/authorize?client_id=xxx&response_type=code&redirect_uri=https%3A%2F%2Flocalhost%3A8888&scope=user-library-read+streaming+playlist-read-private+user-top-read+user-read-playback-state
Enter the URL you were redirected to: https://localhost:8888/?code=.....
ocp_spotify oauth token saved
```

2. Paste your ClientID
3. Copy the Client secret in the Spotify developer site.
4. Copy the URL from the command line and open it in the browser.
5. Choose accept in the Spotify screen; you will be redirected.
6. Copy the redirect URL from the page that is opened (alltough it gives a 'can't connect error')
7. Past the URL in the command line
8. The ocp_spotify oauth token saved at ~/.config/spotipy


## Spotify via OVOS
If you want to make the OVOS device itself a Spotify player, we recommend to install [spotifyd](https://github.com/Spotifyd/spotifyd).

1. Install or build Spotifyd (in this example we install it in /home/ovos/.local/bin/)
2. Create user config named spotifyd.conf in ~/.config/spotifyd, see [example config](https://docs.spotifyd.rs/config/File.html) 
3. Comment username and password to let Spotifyd automatically search for credentials. 
4. Comment also proxy 
5. Set `cache_path` to a writable directory (e.g. `/home/user/.cache/spotifyd`)
6. Set the device_name, device_name = "device_name_in_spotify_connect"
7. Run ~/spotifyd --no-daemon in the command line
8. Connect with any of the official clients to `spotifyd` (which has to be on the same network).
9. Stop `spotifyd`. In the cache directory that you set before, there is now a file called `credentials.json`. Note: in version 0.35 it currently doesn't work, in 0.34 it is.
10. Copy the username of `credentials.json` and uncomment and set the username in spotifyd.conf
11. Start `spotifyd` again with ~/spotifyd --no-daemon, it should now authenticate successfully

To use OVOS commands like pause, play, resume etc:

12. Copy [spotifyd_hooks.py](https://github.com/OpenVoiceOS/ovos-media-plugin-spotify/blob/1dccb60de2750224b4018f3a9c7e532c3c15b760/spotifyd_hooks.py) to your install
13. Run `spotifyd` with 
```
/home/ovos/.local/bin/spotifyd --device-type "speaker" --initial-volume 100 --on-song-change-hook "/home/ovos/.venvs/ovos/bin/python /home/ovos/spotifyd_hooks.py" --no-daemon
```

## Configuration

Edit your user mycroft.conf with any Spotify players you want to expose. The configuration is needed for both situations (OVOS as a player or remote for other devices).
The easiest way is to use the provided `ovos-spotify-autoconfigure` command. 
It will automatically search for all the Spotify clients on the same network.

```bash
$ ovos-spotify-autoconfigure
This script will auto configure ALL spotify devices under your mycroft.conf
        
        SPOTIFY PREMIUM is required!
        
        If you have not yet authenticated your spotify account, run 'ovos-spotify-oauth' first!
        
Found device: OpenVoiceOS-TV

mycroft.conf updated!

# Legacy Audio Service:
{'backends': {'spotify-OpenVoiceOS-TV': {'active': True,
                                         'identifier': 'OpenVoiceOS-TV',
                                         'type': 'ovos_spotify'}}}

# ovos-media Service:
{'audio_players': {'spotify-OpenVoiceOS-TV': {'active': True,
                                              'aliases': ['OpenVoiceOS-TV'],
                                              'identifier': 'OpenVoiceOS-TV',
                                              'module': 'ovos-media-audio-plugin-spotify'}}}
```

### ovos-audio

```javascript
{
  "Audio": {
    "backends": {
      "spotify": {
        "type": "ovos_spotify",
        "identifier": "device_name_in_spotify",
        "active": true
      }
    }
  }
}
```

### ovos-media

> **WARNING**: `ovos-media' has not yet been released, WIP

```javascript
{
 "media": {
    "audio_players": {
        "desk_speaker": {
            "module": "ovos-media-audio-plugin-spotify",
            
            // this needs to be the name of the device on spotify!
            "identifier": "Mark2",

            // users may request specific handlers in the utterance
            // using these aliases
            "aliases": ["office spotify", "office", "desk", "workstation"],

            // deactivate a plugin by setting to false
            "active": true
        }
    }
}
```

## Python usage

If you don't want to use [the companion skill](https://github.com/OpenVoiceOS/skill-ovos-spotify), you can also write your own integrations

```python
s = SpotifyClient()
# pprint(s.query_album("hail and kill by manowar")[1])

from ovos_utils.skills.audioservice import ClassicAudioServiceInterface
from ovos_utils.messagebus import FakeBus

bus = FakeBus()
audio = ClassicAudioServiceInterface(bus)

audio.play("spotify:playlist:37i9dQZF1DX08jcQJXDnEQ")
audio.play(["spotify:track:5P2Ghhv0wFYThHfDQaS0g5",
            "spotify:playlist:37i9dQZF1DX08jcQJXDnEQ"])
time.sleep(5)
audio.pause()
time.sleep(5)

audio.resume()
time.sleep(5)

audio.next()
time.sleep(5)

audio.prev()
time.sleep(5)

print(audio.track_info())
```
