# PySteam

Python interface for Steam Web Chat.

Ported from [node-steamcommunity](https://github.com/DoctorMcKay/node-steamcommunity).
Includes a port of [node-steamid](https://github.com/DoctorMcKay/node-steamid) as well.
Steam APT DOC on [Steam_Web_API](https://partner.steamgames.com/doc/webapi_overview).
Used some ideas from [steam-chat-bot](https://www.npmjs.com/package/steam-chat-bot).

## Formats
Formely i will udes Json.

## Most importend method 
* 

## Packages Required

    pip install requests pycryptodome pyee enum34 pyquery future munch

`py.test` also needs to be installed for testing, if developing.

`ipython` is used for interactivity in the examples.

## Usage

`import` the Steam API

```python
import steamapi
steam = steamapi.steamapi()
```

Define the events you need to respond to with the `pyee` event decorators.

The current events emitted are:

* `chat_persona_state`
    - `steam_id`: instance of [`steamapi.SteamID`](#quick-tangent-steamid) for the [persona](#persona-data) which changed
    - `persona`: the newly updated [persona](#persona-data)
    - `old_persona`: the last [persona](#persona-data) stored
* `chat_message`
    - `sender`: instance of [`steamapi.SteamID`](#quick-tangent-steamid), containing the sender's ID
    - `message`: the message the sender has sent
    - `own`: if the sender of the message was your own account (in that case sender is instead the recipient)
* `chat_typing`
    - `sender`: instance of [`steamapi.SteamID`](#quick-tangent-steamid), containing the typing user's ID

Which can be invoked like so:

```python
@steam.event.on('chat_persona_state')
def chatPersonaStateHandler(steam_id, persona, old_persona):
    pass

@steam.event.on('chat_message')
def chatMessageHandler(sender, text, own):
    # keep in mind that when `own` is true, the sender is infact
    # the recipient of the message, with the actual sender being
    # steam.steam_id (yourself)
    # (you can get your own persona through steam.accountPersona)
    # this only happens when using multiple clients for chat AFAIK
    print steam.chat.friends[str(sender.as_64)].name + ':', text
    if text.lower() == "ping":
        steam.chat.send_message(sender, "pong")

@steam.event.on('chat_typing')
def chatTypingHandler(steam_id):
    pass
```

Logging in can be achieved through this code snippet:

```python
status = steam.login(username=username, password=password)
while status != steamapi.enums.LoginStatus.LoginSuccessful:
    if status == steamapi.enums.LoginStatus.TwoFactor:
        token = input("Two-factor Token: ")
        status = steam.retry(twofactor=token)
    elif status == steamapi.enums.LoginStatus.SteamGuard:
        steamguard = input("SteamGuard Code: ")
        status = steam.retry(steamguard=steamguard)
    elif status == steamapi.enums.LoginStatus.Captcha:
        captcha = input("CAPTCHA: ")
        status = steam.retry(captcha=captcha)
```

(replace `input` with `raw_input` on Python 2)

Status will be `SteamAPI.enums.LoginStatus.LoginSuccessful` once you have logged in.
From here, you can call `steam.chat.login()` to initiate a connection with the chat API.

After you have logged in, `steam.chat.friends` will be populated with the [persona](#persona-data) of the users on your friends list, as a dict with their `SteamID64` as the key.

Nothing will be needed past this in terms of Steam connection.
Once you have finished doing what you're doing, call `steam.chat.logoff()` to gracefully disconnect from the Steam chat servers.


## Quick Tangent: SteamID

I have also ported `node-steamid` to Python. It can be initialized with:

* `STEAM_0:0:34589227` (Steam ID2)
* `[U:1:69178454]` (Steam ID3)
* `76561198029444182` (Steam64 ID)
* `69178454` (Steam32 ID)

In these examples, `sid = SteamID('STEAM_0:0:34589227')`
It exposes these methods:

* `sid.is_valid()`
* `SteamID.from_account_id(account_id)`

And these properties:

* `sid.accountid`
* `sid.type`
* `sid.instance`
* `sid.universe`

Rendered Forms (properties):

* `sid.as_64` (e.g. `76561198029444182`)
* `sid.as_32` (e.g. `69178454`)
* `sid.as_steam2` (e.g. `STEAM_0:0:34589227`)
* `sid.as_steam3` (e.g. `[U:1:69178454]`)

## Persona Data

A **persona** is a dictionary of user info, usually like so:

```python
{
    'avatar_hash': 'eed791eee6442a03a5acb2470289ee4c0f5995aa',
    'ingame': False,
    'ingame_app_id': None,
    'ingame_name': None,
    'name': 'sgfc.yuuna',
    'nickname': None,
    'state': <steamapi.enums.PersonaState.Online: 1>,
    'state_flags': <steamapi.enums.PersonaStateFlag.OnlineUsingWeb: 256>,
    'steam_id': SteamID.SteamID('76561198047347491')
}
```

* `avatar_hash` is the hash of the user's Steam avatar.
    * Use `steamapi.utils.url_avatar(avatar_hash)` to get the full URL.
* `ingame` is a boolean describing if the user is currently playing a game on Steam or not.
* `ingame_app_id` is the Steam AppID of the game currently being played if `inGame` is true, else it is `None`.
* `ingame_name` is the name of the game currently being played if `inGame` is true, else it is `None`.
* `name` is the steam display name set by the user.
* `nickname` is your user-set nickname.
* `state` is the user's online status on Steam. One of:
    * `steamapi.enums.PersonaState.Offline`
    * `steamapi.enums.PersonaState.Online`
    * `steamapi.enums.PersonaState.Busy`
    * `steamapi.enums.PersonaState.Away`
    * `steamapi.enums.PersonaState.Snooze`
    * `steamapi.enums.PersonaState.LookingToTrade`
    * `steamapi.enums.PersonaState.LookingToPlay`
* `state_flags` describes if the user is using a special mode of Steam in chat.
    * `steamapi.enums.PersonaStateFlag.HasRichPresence`
    * `steamapi.enums.PersonaStateFlag.InJoinableGame`
    * `steamapi.enums.PersonaStateFlag.OnlineUsingWeb`
    * `steamapi.enums.PersonaStateFlag.OnlineUsingMobile`
    * `steamapi.enums.PersonaStateFlag.OnlineUsingBigPicture`
* `steam_id` is an instance of [`steamapi.SteamID`](#quick-tangent-steamid).


## Sending Messages

Messages can be sent via the `steam.chat.send_message` method.

```python
steam.chat.send_message(SteamID64, message)
```
