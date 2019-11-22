A sexy achievement file parser with real-time notification.<br />
View every achievement earned on your PC whether it's coming from Steam, a Steam emulator, and more.<br />
To see the full list of what this app can import please see the [**Compatibility**](https://github.com/xan105/Achievement-Watcher#compatibility-) section.

<table >
<tr>
<td align="left"><img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/home.png" width="400px"></td>
<td align="left"><img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/ach_view.png" width="400px"></td>
</tr>
</table>

The original idea behind this app was that some steam emulators generate a text file where your unlocked achievements are stored. 
But they aren't very friendly to know which is which, here is an example :
```ini
[NEW_ACHIEVEMENT_1_1]
Achieved=1
CurProgress=0
MaxProgress=0
UnlockTime=0000000000
[SteamAchievements]
00000=NEW_ACHIEVEMENT_1_1
Count=1
```
So which achievement is NEW_ACHIEVEMENT_1_1 ? You'll have to ask the steam API or look online in a site like the steamdb to find out.
So let's just do that automagically :)

Notification on achievement unlocking
==========================================

Not as sexy as a directX Overlay but it's the next best thing.<br />
Display a Windows toast notification when you unlock an achievement.<br />
⚠️ **Please verify your Windows notification and focus assistant settings for the toast to work properly**.<br />
You can test notification in Settings > Debug to make sure your system is correctly configured.

<p align="center">
  <img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/live.gif">
</p>

There might be a slight delay between the event and the display of the notification as running powershell and loading a remote image can take a few seconds in some cases.<br />

Game must be set to Window borderless for the notification to be rendered on top of it.<br />

If you have enabled the *souvenir* option a screenshot will be taken<br />
and saved in your pictures folder `"Pictures\[Game Name]\[Game Name] - [Achievement Name].png"`<br />

### 🚑 Not seeing any toast notification ? Quick fix :
- Try to set your game to Window borderless.
- Try to disable the automatic game **and** fullscreen rule in focus assistant (Win10)<br/>
  or set them to priority and make sure that the UWP appID you are using is in your priority list (By default the Xbox appID(s) used by this app are in it).
- Try to set checkIfProcessIsRunning to false in `%AppData%\Achievement Watcher\cfg\options.ini`

Windows 8.1 : Don't forget quiet hours.<br />
Windows 10 >= 1903 : New focus assist auto rule for fullscreen app set to alarm only by default prevents the notification from working out of the box.

The process `watchdog.exe` is the one doing all the work so make sure it is running.

Not all games are supported, please see the [**compatibility**](https://github.com/xan105/Achievement-Watcher#compatibility-) section below.

<hr>

You can also display a notification with:
  - Websocket
  - Growl Notification Transport Protocol (GNTP)

### Websocket

Endpoint: `ws://localhost:8082`

You can for example use this to create your own notification in an OBS browser source and the like for your streaming needs.

Achievement data are broadcasted to all connected websocket clients.<br />
There is a 30sec ping/pong but normally it's handled by the browser so you shouldn't have to add code for it.<br />

To help you there is a test command to send a dummy valid notification.
```js
//Example

const ws = new WebSocket("ws://localhost:8082");
ws.onopen = (evt) => { 
  ws.send(JSON.stringify({cmd:"test"})); //dummy is only send to the client making the request
  
  ws.send(JSON.stringify({
    cmd:"test",
    broadcast: true
 })); //use broadcast option if you need to send the dummy to all connected clients
  
};
ws.onmessage = (evt) => {
  console.log(JSON.parse(evt.data)); //JSON string
  /* Output:
    {
      appID: steam appID,
      title: game name,
      id: achievement id,
      message: achievement title,
      description: achievement description if any,
      icon: unlocked icon url,
      time: timestamp from the Steam emu,
      progress: //if it's a progress and not an unlocked achievement; 
                //Otherwise this property is not sent at all
      { 
          current: current progress,
          max: max progress value
      }
    }  
  */
  ws.close();
};

```

💡 Yes ! You could use this to create a DirectX Overlay by using a DirectX hook which will use Chromium Embedded Framework (such as [momo5502/gameoverlay](https://github.com/momo5502/gameoverlay)) to load some js code to consume data from this websocket.<br />
I successfully done it but only a few DirectX 9 games weren't crashing and since my C++ skills are not up for the task.<br />
I wouldn't mind some help if you know your stuff.    
    
### GNTP

Endpoint: `localhost:23053`

Recommended gntp client is Growl for Windows (despite it being discontinued) [Mirror download link](https://github.com/xan105/Achievement-Watcher/releases/download/1.2.3/Growl.7z)

**Since Windows 7 doesn't have toast notification you can use Growl for Windows to get toast like notification**.<br />

To customize the look of the toast please kindly see your gntp client's options.<br />
If you are looking for the Achievement Watcher notification sounds there are in `%windir%\Media` (Achievement___.wav)
  
Compatibility :
================

|Emulator/Client|Supported|Unlock Time|Ach Progress|Notification|
|--------|---------|-----------|------------|------------|
|Codex| Yes | Yes | Yes, if available | Yes |
|Goldberg Steam Emu| Yes | Yes | No | Yes |
|Hoodlum| Via user custom dir| Yes | No | Yes
|DARKSiDERS| Via user custom dir| Yes | No | Yes
|Skidrow| Yes | No | No | Yes |
|ALI213| Via user custom dir | Yes | No | Yes |
|RLD!| Yes | Yes | No | Yes (on game exit)
|GreenLumaReborn| Yes | No | No | No |
|SmartSteamEmu| [Via this plugin](https://github.com/xan105/Achievement-Watcher/releases/download/1.1.1/SSE_userstatswrapper.rar) | Yes | No | Yes |
|Legit Steam Client| Yes but your Steam profile must be public | Yes | No | Steam overlay does it already | 
|RPCS3 (PS3) | Via user custom dir | No | N/A | RPCS3 does it already|  
|LumaPlay (Uplay) | Yes | No | No | No |


### Steam Emulator
By default the following locations will be scanned for the files generated by steam emulators :
```
- %PUBLIC%\Documents\Steam\CODEX
- %appdata%\Steam\CODEX
- %ProgramData%\Steam\*\
- %localappdata%\SKIDROW
- %DOCUMENTS%\SKIDROW
- %appdata%\SmartSteamEmu
- %appdata%\Goldberg SteamEmu Saves
```

You can add your own folder in the app, just make sure that you select a folder which contains appid folder(s) :<br/>
 ```
 |___ Custom dir
      |___ 480 
      |___ 220 
 ```
NB: To enable notification on a custom folder you need to click the bell icon next to it. 
 
For Steam emulators that don't have a default folder (eg: ALI213) choose the dir where their cfg .ini file is; <br>
The app will then parse it and look for achievement data file from the chosen location.
 
⚠️ Green Luma Reborn: only if the reg key `"SkipStatsAndAchievements"` is set to `dword:00000000` for that APPID.

### Legit Steam
You can choose to view none (default) / only installed / all owned Steam games.<br/>
Ach. are updated based on files timestamp in `STEAM\appcache\stats`<br/>

⚠️ This feature requires that Steam is installed and your Steam Profile is set to `Public`.<br/>

<p align="center">
<img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/steam_privacy.png">
</p>

Due to the server rate limit if you 've a huge Steam library it might not get all your games in one go.<br/>
If you are using your own steam web api key (see **Steam Web API Key** section below), this doesn't concern you.

### RPCS3 Playstation 3 Emulator
Please add a folder in the app where `rpcs3.exe` is located. The app will then look for ~~achievement~~ trophies for every game and every ps3 user.<br/>
Note that `TROPCONF.SFM` is language specific; So for PS3 games, trophies will be in the language you are playing with.<br/>
As of this writing there is no unlock time : the trophies unlocked in a PS3 that has never been connected online doesn't contains timestamps.

### LumaPlay
Since there is no public API to get a Uplay game achievements info as of this writing there are limitations: <br/>
Uplay client must be installed in order to try to get the game's info from its cache.<br/> 
To have the game info in the Uplay client cache you **don't** need to install the game but you need to have at least seen the achievement listing page of the game once in the Uplay client.<br/>
This app will keep and send the data to a remote server to build its own cache, when the server has the game info Uplay client is no longer required as the app will fetch the data from said server. <br/>
Therefore with time only newest game would require Uplay client in theory. <br/>

Options
=======
Options are stored in ```%AppData%\Achievement Watcher\cfg\options.ini``` but most of them are configurable via the GUI<br />

### [achievement]
- lang<br />
  default to user locale<br />
  Both UI and data from Steam.<br />
  
- showHidden<br />
  default to false<br />
  Wether or not show hidden achievements if any.<br />
  
- mergeDuplicate<br />
  default to true<br />
  Try to merge multiple achievement source for the same game.<br />
  
- timeMergeRecentFirst<br/>
  default to false<br />
  When merging duplicates, show the most recent timestamp instead of the oldest.
  
- hideZero<br />
  default to false<br />
  Hide 0% Game.<br />
  
- legitSteam<br />
  default to 0<br />
  Steam games : (0) none / (1) installed / (2) owned.<br />
  
### [notification]

- notify<br />
  default to true<br />
  Notify on achievement unlocking if possible. <br />
  (`AchievementWatcher.exe` doesn't need to be running for this, but `watchdog.exe` does).<br />
  
- powershell <br />
  default to true<br />
  Use powershell to create a Windows 8-10 toast notification.<br />
  
- gntp <br />
  default to true<br />
  Send a gntp@localhost:23053 if available.<br />
  
- souvenir<br />
  default to true<br />
  Take a screenshot when you unlock an achievement in<br />
  `"Pictures\[Game Name]\[Game Name] - [Achievement Name].png"`<br />
  
- toastSouvenir<br />
  default to 0<br />
  Display souvenir screenshot inside the toast (Win10 only).<br />
  (0) disable / (1) header (image crop) / (2) footer (image resized to fit) <br />
  <br />
  Example:
  
  <table >
  <tr>
  <td align="center">header</td>
  <td align="center">footer</td>
  </tr> 
  <tr>
  <td align="left"><img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/toastedSouvenir_header.gif" width="400px"></td>
  <td align="left"><img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/toastedSouvenir_footer.gif" width="400px"></td>
  </tr>
  </table>
  
  Both will show the screenshot within their toast in the action center if there is enough space.<br />
  Otherwise there will be an arrow to show/hide (collapse).<br />
  
- showDesc<br />
  default to false<br />
  Show achievement description if any.<br />
  
- customToastAudio<br />
  default to 1<br />
  Specifies the sound to play when a toast notification is displayed.<br />
  (0) disable-muted / (1) System default / (2) Custom sound specified by user<br />
  
- rumble<br />
  default to true<br />
  Vibrates first xinput controller when unlocking an achievement.<br />
  
- notifyOnProgress<br />
  default to true<br />
  Notify on achievement progress when possible.<br />
  
  <p align="center">
    <img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/ach_progress.png" width="800px">
  </p>
  
### [notification_advanced]

👮 Change these values only if you know what you are doing.<br />

- timeTreshold<br />
  default to 10 (sec)<br />
  Amount of sec an achievement is considered achieved from its timestamp value before being discarded.<br />
  
- checkIfProcessIsRunning<br />
  default to true<br />
  When an achievement file is modified; Wether to check or not if the corresponding game is running and responding.<br />
  <br />
  Both options are mainly there to mitigate false positive.<br />
  
- tick<br />
  default to 600 (ms)<br />
  Ignore file modification within specified timeframe to prevent spam of notification when a game triggers multiple file write at the same time.<br />
  Set it to 0 to disable this feature.<br />
  
- appID<br />
  if not set, default to Xbox Game Bar if available otherwise to Xbox App<br />
  Notification appID ([Application User Model ID](https://docs.microsoft.com/fr-fr/windows/desktop/shell/appids)).<br />
  Example: 
  
  |Name| AppID |
  |----|-------|
  |Xbox Game Bar|Microsoft.XboxGamingOverlay_8wekyb3d8bbwe!App |
  |Xbox App| Microsoft.XboxApp_8wekyb3d8bbwe!Microsoft.XboxApp |
  |Xbox App (Win 8)| microsoft.XboxLIVEGames_8wekyb3d8bbwe!Microsoft.XboxLIVEGames |
  
  ⚠️ You need to use a UWP AppID otherwise you won't be able to remotely load ach. img.
  
Steam Web API Key
=================
Some use of the Steam Web API to fetch data from Steam requires the use of an API Key.<br />
If you leave the field blank in the settings section, it will automagically fetch said data.<br />

<p align="center">
<img src="https://github.com/xan105/Achievement-Watcher/raw/master/screenshot/settings.png" width="600px">
</p>

An example of a server that feeds you the data is provided within this repo.<br />
This service is not guarantee over time and is solely provided for your own convenience.<br />
If you experience any issues please use your own Steam Web API key.<br />              
                
You can acquire one [by filling out this form](https://steamcommunity.com/dev/apikey).<br />
Use of the APIs also requires that you agree to the [Steam API Terms of Use](https://steamcommunity.com/dev/apiterms).<br />

Command Line Args | URI Scheme
==============================

Args:<br />
`--appid xxx [--name yyy]`<br />

URI:<br />
`ach:--appid xxx [--name yyy]`<br />

xxx is a steam appid<br />
yyy is an optional steam ach id name<br />

After the loading directly display the specified game.<br />
And if specified highlight an achievement.

NB: This is what the toast notification uses in order to be clickable and open the game page highlighting the unlocked achievement.

Translation Help
================

I do my best to translate everything for every supported language by Steam, but it's rather difficult and I don't speak that much languages.
Fluent in another language ? Any help to add/modify/improve would be greatly appreciated.

More details [here](https://github.com/xan105/achievement-watcher/tree/master/app/locale)

Auto-Update
===========

This software auto update itself via Windows scheduled tasks.
There are .cmd files in the root directory to create, delete and manually run the tasks.

File cache & Logs
=================
in ```%AppData%\Achievement Watcher```

How to build
============
This repo uses [Git LFS](https://git-lfs.github.com/).<br/>
You will need node.js (>= 12.x), golang (>= 1.12) both with the same arch: in x86 **or** x64.<br/>
Innosetup 5 unicode with preprocessor and [Inno Download Plugin](https://mitrichsoftware.wordpress.com/inno-setup-tools/inno-download-plugin/)<br/>
Golang cgo requires a gcc compiler installed and set in PATH (recommended : http://tdm-gcc.tdragon.net/download).<br/>

For node you globally need asar, json and pkg :<br/>
```
npm install -g asar json pkg
```
NB: If pkg fetches a version different than `%userprofile%\.pkg-cache\v2.6\fetched-v12.2.0-win-x64` or you are building this in x86.<br/>
You need to update `service\rcedit-updater.cmd` and `service\rcedit-watchdog.cmd` with the correct one.

If you build this in x86 remove these lines in `setup\AchievementWatcher.iss`: <br/>
```
60 ArchitecturesInstallIn64BitMode=x64
61 ArchitecturesAllowed=x64
```

Use `buildme.cmd` in the root folder to build.

NB: Innosetup is expected to be installed in `C:\Program Files (x86)\Inno Setup 5` if that is not the case then update `buildme.cmd` with the correct path.

Legal
=====
Software provided here is to be use at your own risk. This is provided as is without any express or implied warranty.<br />
In no event or circumstances will the authors or company be held liable for any damage to yourself or your computer that may arise from the installation or use of the free software aswell as his documentation that is provided on this website.<br />
And for anything that may occur as a result of your use, or inability to use the materials provided via this website.<br />

Software provided here is purely for informational purposes and does not provide nor encourage illegal access to copyrighted material.<br />

Software provided here is not affiliated nor associated with Steam, © Valve Corporation and data from its API is provided as is without any express or implied warranty.<br />

Software provided here is not affiliated nor associated with Uplay, © Ubisoft and data from its API is provided as is without any express or implied warranty.<br />

Software provided here is not affiliated nor associated with any cracking scene groups.<br />

Other trademarks, copyright are the property of their respective owners. No copyright or trademark infringement is intended by using third-party resources. Except where otherwise specified, the contents of this project is subject to copyright.<br />
