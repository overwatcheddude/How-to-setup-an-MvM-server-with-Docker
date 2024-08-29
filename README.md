This is a simple guide on how to start an MvM server using Docker. This also works with Podman.

## 1\. Generating an SRCDS Token

`SRCDS_TOKEN` is required if you want your server to be visible on the server browser. Follow the below steps to generate one:

1. Open the following link:  
    [https://steamcommunity.com/dev/managegameservers](<https://steamcommunity.com/dev/managegameservers>)

2. In the App ID field, type the following:  
    440

3. In the Memo field, type whatever you see fit. You may use the following:  
    MvM Server

4. Click on the "Create" button.

5. Copy the login token.

## 2\. Docker

There are plenty of images that can be picked, which can be found here:  
[https://github.com/CM2Walki/TF2?tab=readme-ov-file#image-variants](<https://github.com/CM2Walki/TF2?tab=readme-ov-file#image-variants>)

For this guide, we will be going with the image that comes with SourceMod pre-installed.

Pull the latest image:

```bash
docker pull cm2network/tf2:sourcemod
```

Start and run the container, make sure to replace `{YOURTOKEN}` with the token you generated and `{YOURPASSWORD}` with a secure password:

```bash
docker run --net=host --name=my-mvm-server -e SRCDS_TOKEN="{YOURTOKEN}" -e SRCDS_RCONPW="{YOURPASSWORD}" -e SRCDS_PW="" -e SRCDS_STARTMAP="mvm_decoy" -e SRCDS_MAXPLAYERS=32 cm2network/tf2:sourcemod
```

Here's a breakdown for the command:

- `docker run`: This command creates and starts a new container.
- `--net=host`: The container shares the host's network stack, meaning it directly uses the host's IP address and ports.
- `--name=my-mvm-server`: This names the container "my-mvm-server". You may change its name to whatever you see it.
- `-e SRCDS_TOKEN={YOURTOKEN}`: This sets an environment variable `SRCDS_TOKEN` with a value of `{YOURTOKEN}`. This token is necessary to authenticate your server with Valve. Ensure to replace it with your own token.
- `-e SRCDS_RCONPW={YOURPASSWORD}`: This sets an environment variable `SRCDS_RCONPW` with a value of `{YOURPASSWORD}`. This is the password for remote console access to the server. Ensure to replace it with a secure password.
- `-e SRCDS_PW=""`: This sets the environment variable `SRCDS_PW` to an empty string, meaning no password is required for players to join the server.
- `-e SRCDS_STARTMAP="mvm_decoy"`: This sets the initial map to "mvm\_decoy" when the server starts. You may replaced it with any other MvM map if you want.
- `-e SRCDS_MAXPLAYERS=32`: MvM bots take up player slots, so you need to set the max player to at least 32.
- `cm2network/tf2:sourcemod`: This specifies the Docker image to use, which in this case is a TF2 server image with SourceMod pre-installed.

You can find other environment variables here:  
[https://github.com/CM2Walki/TF2?tab=readme-ov-file#environment-variables](<https://github.com/CM2Walki/TF2?tab=readme-ov-file#environment-variables>)

After starting the container successfully, wait for the server to update. It might take some time.

Once the server update is complete, launch TF2, open the server browser, click on the "LAN" tab, if everything went well then you will be able to see and connect to your server. However, there are still things need to be done, so continue reading the guide.

## 3\. Map rotation

To add stock MvM maps to your map cycle, run the following command:

```bash
docker exec -it my-mvm-server nano /home/steam/tf-dedicated/tf/cfg/mapcycle.txt
```

Add the following content to `mapcycle.txt`:

```
mvm_bigrock
mvm_coaltown
mvm_decoy
mvm_ghost_town
mvm_mannhattan
mvm_mannworks
mvm_rottenburg
```

## 4\. Admin privileges

Since the server comes with SourceMod, it is a good idea to set yourself as an admin in order to access admin-only features.

### 4\.1. Finding your Steam ID

1. Open the following website:  
    [https://steamdb.info/calculator/](<https://steamdb.info/calculator/>)

2. In the blank field, enter your Steam community profile URL, for example:  
    [https://steamcommunity.com/id/kisats/](<https://steamcommunity.com/id/kisats/>)

3. Click on the "Get disappointed in your life" button.

4. Copy the "Steam2 ID" value.

### 4\.2. Becoming an Admin

To edit the admin list, run the following command:

```bash
docker exec -it my-mvm-server nano /home/steam/tf-dedicated/tf/addons/sourcemod/configs/admins_simple.ini
```

At the end of the file, add the following line. Ensure to replace `YOUR_STEAM_ID_HERE` with the Steam ID you copied previously:

```
"YOUR_STEAM_ID_HERE"  "99:z"
```

## 5\. Changing server.cfg

The `server.cfg` file that comes with the image is targeted towards normal TF2 matches. It is important to change it so it fits MvM requirements. To do so, follow the steps below:

Remove the existing `server.cfg` file:

```bash
docker exec -it my-mvm-server rm /home/steam/tf-dedicated/tf/cfg/server.cfg
```

Create a new `server.cfg` file:

```bash
docker exec -it my-mvm-server nano /home/steam/tf-dedicated/tf/cfg/server.cfg
```

In the `server.cfg` file, insert the following content. Ensure to replace the values of `hostname`, `rcon_password`, `sv_contact` and `sv_region`:

```
// Change the following:
hostname my-mvm-server // This is the server name. It will be shown on the server browser.
rcon_password ChangeMe // Enter a secure password
sv_contact test@gmail.com // Type your email address
sv_region 255 // 0: US - East, 1: US - West, 2: South America, 3: Europe, 4: Asia, 5: Australia, 6: Middle East, 7: Africa, 255: World (Default)

// General MVM Settings //
tf_mm_servermode 1 //puts the server into mvm matchmaking
tf_mm_strict 0	//allows players to join through matchmaking and server browsing
tf_mm_match_size_mvm 6 //minimum players needed in matchmaking before connecting
tf_mvm_min_players_to_start 1 //default 3
tf_mvm_respec_enabled 1 //allow refunds
tf_mvm_respec_limit 0 //total allowed refunds; 0 - unlimited
tf_mvm_respec_credit_goal 2000 //if respec_limit is not 0, then the total number of credits needed to earn a refund
tf_mvm_skill 3 // 1 - easy; 3 - medium (default); 5 - hard
tf_mvm_disconnect_on_victory 0 //disconnect players on victory
tf_mvm_victory_reset_time 60 //seconds to wait after victory before changing map

// Set to lock per-frame time elapse
host_framerate 0
// Set the pause state of the server
setpause 0
// Control where the client gets content from 
// 0 = anywhere, 1 = anywhere listed in white list, 2 = steam official content only
sv_pure 0
// Is the server pausable
sv_pausable 0
// Type of server 0=internet 1=lan
sv_lan 0
// Whether the server enforces file consistency for critical files
sv_consistency 1
// Collect CPU usage stats
sv_stats 1
//Tags
//sv_tags <your Tags>
//Server Player Password
// NOTE: if you have a password, your MvM server will not be allowed in Match Making.
//sv_password <your server password>

// Execute Banned Users //
exec banned_user.cfg
exec banned_ip.cfg
writeid
writeip

// Rcon Settings //

// Number of minutes to ban users who fail rcon authentication
sv_rcon_banpenalty 1440
// Max number of times a user can fail rcon authentication before being banned
sv_rcon_maxfailures 5

// Log Settings //

// Enables logging to file, console, and udp < on | off >.
log on
// Log server information to only one file.
sv_log_onefile 1
// Log server information in the log file.
sv_logfile 1
// Log server bans in the server logs.
sv_logbans 1
// Echo log information to the console.
sv_logecho 1

// Rate Settings //

// Frame rate limiter
fps_max 600
// Min bandwidth rate allowed on server, 0 == unlimited
sv_minrate 0
// Max bandwidth rate allowed on server, 0 == unlimited
sv_maxrate 20000
// Minimum updates per second that the server will allow
sv_minupdaterate 66
// Maximum updates per second that the server will allow
sv_maxupdaterate 66

// Download Settings //

// Allow clients to upload customizations files
sv_allowupload 1
// Allow clients to download files
sv_allowdownload 1
// Maximum allowed file size for uploading in MB
net_maxfilesize 64

//VOTING!//

sv_allow_votes 1
sv_vote_allow_spectators 0
sv_vote_failure_timer 120 //(default 300 = 5 minutes)

// REGULAR GAME VOTES //
//Enable Scramble Vote
sv_vote_issue_scramble_teams_allowed 0
//Enable Restart Game
sv_vote_issue_restart_game_allowed 1
//Enable NextLevel Vote
sv_vote_issue_nextlevel_allowed 1
//Enable Kick vote
sv_vote_issue_kick_allowed 1
//Kick Duration (0 for no ban time, non-0 for minutes to ban)
sv_vote_kick_ban_duration 10

// MVM VOTES //
//Enable Kick vote
sv_vote_issue_kick_allowed_mvm 1
//Enable changelevel vote
sv_vote_issue_changelevel_allowed_mvm 1
//Enable restart map vote
sv_vote_issue_restart_game_allowed_mvm 1
//Enable classlimits vote
sv_vote_issue_classlimits_allowed_mvm 0
//Enable classlimit max count vote
sv_vote_issue_classlimits_max_mvm 2 //(default 2)
//Enable kick players that haven't connected yet but passed certain time threshold
sv_vote_issue_kick_min_connect_time_mvm 0 //(default 0 is enabled)
//Vote timer cooldown
sv_vote_failure_timer_mvm 120 //(default 120 = 2 minutes)
//Allow change difficulty vote
sv_vote_issue_mvm_challenge_allowed 1

// Round and Game Times //

//Wait for Players
mp_waitingforplayers_cancel 1
// Enable timers to wait between rounds. WARNING: Setting this to 0 has been known to cause a bug with setup times lasting 5:20 (5 minutes 20 seconds) on some servers!
mp_enableroundwaittime 1
// Time after round win until round restarts
mp_bonusroundtime 10
// If non-zero, the current round will restart in the specified number of seconds
mp_restartround 0
//Enable sudden death
mp_stalemate_enable 0
// Timelimit (in seconds) of the stalemate round.
mp_stalemate_timelimit 300
// game time per map in minutes
mp_timelimit 60
//Max Round Wins
mp_winlimit 0
//Disable Respawn Times
mp_disable_respawn_times 0 //(default 0; 1 allows near instant respawns)
// Overrides the max players reported to prospective clients
sv_visiblemaxplayers 6
// Maximum number of rounds to play before server changes maps
mp_maxrounds 0

// Client CVARS //

// Restricts spectator modes for dead players
mp_forcecamera 1
// toggles whether the server allows spectator mode or not
mp_allowspectators 1
// toggles footstep sounds
mp_footsteps 1
// toggles game cheats
sv_cheats 0
// After this many seconds without a message from a client, the client is dropped
sv_timeout 900
// Maximum time a player is allowed to be idle (in minutes), made this and sv_timeout equal same time?
mp_idlemaxtime 9
// Deals with idle players 1=send to spectator 2=kick
mp_idledealmethod 1
// time (seconds) between decal sprays
decalfrequency 10
//Overtime Nagging
tf_overtime_nag 1

// Communications //

// enable voice communications
sv_voiceenable 1
// Players can hear all other players, no team restrictions 0=off 1=on
sv_alltalk 0
// amount of time players can chat after the game is over
mp_chattime 10
// enable holiday modes: 0none,1birthday,2halloween,3birthday
//tf_forced_holiday 0
```

## 6\. Checklist

To ensure that everything went well, check the following things:

- The server is visible on the server browser.

- The maps and missions can be changed through voting.

- The admin commands can be accessed by typing `!sm_admin` in the chat.

- When pressing F4, the wave starts and the robots spawn as expected.

## 7\. Credits

- Thanks to CM2Walki for providing [TF2 Docker images](<https://github.com/CM2Walki/TF2> "TF2 Docker images").

- Thanks to Jamillia for providing [server.cfg](<https://steamcommunity.com/discussions/forum/13/612823460277729158/> "server.cfg")
