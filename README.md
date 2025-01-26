<p align="center">
  <img src="extras/dockcheck_logo_by_booYah187.png" width="160" title="dockcheck">
</p>
<p align="center">
  <img src="https://img.shields.io/badge/bash-4.3-green?style=flat-square&logo=gnubash" alt="bash">
  <a href="https://www.gnu.org/licenses/gpl-3.0.html"><img src="https://img.shields.io/badge/license-GPLv3-red?style=flat-square" alt="GPLv3"></a>
  <img src="https://img.shields.io/github/v/tag/mag37/dockcheck?style=flat-square&label=release" alt="release">
  <a href="https://ko-fi.com/mag37"><img src="https://img.shields.io/badge/-Ko--fi-grey?style=flat-square&logo=Ko-fi" alt="Buy me a Coffee"></a>
  <a href="https://liberapay.com/user-bin-rob/donate"><img src="https://img.shields.io/badge/-LiberaPay-grey?style=flat-square&logo=liberapay" alt="LiberaPay"></a>
  <a href="https://github.com/sponsors/mag37"><img src="https://img.shields.io/badge/-Sponsor-grey?style=flat-square&logo=github" alt="Github Sponsor"></a>
</p>

<h3 align="center">CLI tool to automate docker image updates. <br>No <b>pre-pull</b>, selective, optional notifications and prune when done.</h3>
<h2 align="center">Now with simple notification integrations!</h2>
<h4 align="center">With features like excluding specific containers, custom container labels, auto-prune when done and more.</h4>
<h4 align="center">Also see the fresh Podman fork <a href="https://github.com/sudo-kraken/podcheck">sudo-kraken/podcheck</a>!</h4>

___
## :bell: Changelog

- **v0.5.2.1**: Rewrite of dependency downloads, jq can be installed with package manager or static binary.
- **v0.5.1**: DEPENDENCY WARNING: now requires **jq**. + Upstreaming changes from [sudo-kraken/podcheck](https://github.com/sudo-kraken/podcheck)
- **v0.5.0**: Rewritten notify logic - all templates are adjusted and should be migrated!
    - Copy the custom settings from your current template to the new version of the same template.
    - Look into, copy and customize the `urls.list` file if that's of interest.
    - Other changes: 
        - Added Discord notify template.
        - Verbosity changed of `regctl`.
- **v0.4.9**: Added a function to enrich the notify-message with release note URLs. See [Release notes addon](https://github.com/mag37/dockcheck#date-release-notes-addon-to-notifications)
- **v0.4.8**: Rewrote prune logic to not prompt with options `-a|-y` or `-n`. Auto prune with `-p`.
___


![](extras/example.png)

## :mag_right: `dockcheck.sh`
```
$ ./dockcheck.sh -h
Syntax:     dockcheck.sh [OPTION] [part of name to filter]
Example:    dockcheck.sh -y -d 10 -e nextcloud,heimdall

Options:"
-a|y   Automatic updates, without interaction.
-c     Exports metrics as prom file for the prometheus node_exporter. Provide the collector textfile directory.
-d N   Only update to new images that are N+ days old. Lists too recent with +prefix and age. 2xSlower.
-e X   Exclude containers, separated by comma.
-f     Force stack restart after update. Caution: restarts once for every updated container within stack.
-h     Print this Help.
-i     Inform - send a preconfigured notification.
-l     Only update if label is set. See readme.
-m     Monochrome mode, no printf color codes.
-n     No updates, only checking availability.
-p     Auto-Prune dangling images after update.
-r     Allow updating images for docker run, wont update the container.
-s     Include stopped containers in the check. (Logic: docker ps -a).
-t     Set a timeout (in seconds) per container for registry checkups, 10 is default.
-v     Prints current version.
```

### Basic example:
```
$ ./dockcheck.sh
. . .
Containers on latest version:
glances
homer

Containers with updates available:
1) adguardhome
2) syncthing
3) whoogle-search

Choose what containers to update:
Enter number(s) separated by comma, [a] for all - [q] to quit:
```
Then it proceeds to run `pull` and `up -d` on every container with updates.  
After the updates are complete, you'll get prompted if you'd like to prune dangling images.

___

## :nut_and_bolt: Dependencies
- Running docker (duh) and compose, either standalone or plugin. (see [Podman fork](https://github.com/sudo-kraken/podcheck)  
- Bash shell or compatible shell of at least v4.3
- [jq](https://github.com/jqlang/jq) 
  - User will be prompted to install with package manager or download static binary.
- [regclient/regctl](https://github.com/regclient/regclient) (Licensed under [Apache-2.0 License](http://www.apache.org/licenses/LICENSE-2.0))  
  - User will be prompted to download `regctl` if not in `PATH` or `PWD`.  
  - regctl requires `amd64/arm64` - see [workaround](#roller_coaster-workaround-for-non-amd64--arm64) if other architecture is used.

## :tent: Install Instructions
Download the script to a directory in **PATH**, I'd suggest using `~/.local/bin` as that's usually in **PATH**.
```sh
# basic example with curl:
curl -L https://raw.githubusercontent.com/mag37/dockcheck/main/dockcheck.sh -o ~/.local/bin/dockcheck.sh
chmod +x ~/.local/bin/dockcheck.sh

# or oneliner with wget:
wget -O ~/.local/bin/dockcheck.sh "https://raw.githubusercontent.com/mag37/dockcheck/main/dockcheck.sh" && chmod +x ~/.local/bin/dockcheck.sh
```
Then call the script anywhere with just `dockcheck.sh`.
Add preferred `notify.sh`-template to the same directory - this will not be touched by the scripts self-update function.


## :loudspeaker: Notifications
Trigger with the `-i` flag.  
Run it scheduled with `-ni` to only get notified when there's updates available!  

Use a `notify_X.sh` template file from the **notify_templates** directory, copy it to `notify.sh` alongside the script, modify it to your needs! (notify.sh is added to .gitignore)  
**Current templates:**
- Synology [DSM](https://www.synology.com/en-global/dsm)
- Email with [mSMTP](https://wiki.debian.org/msmtp) (or deprecated alternative [sSMTP](https://wiki.debian.org/sSMTP))
- Apprise (with it's [multitude](https://github.com/caronc/apprise#supported-notifications) of notifications)
  - both native [caronc/apprise](https://github.com/caronc/apprise) and the standalone [linuxserver/docker-apprise-api](https://github.com/linuxserver/docker-apprise-api)
  - Read the [QuickStart](extras/apprise_quickstart.md)
- [ntfy.sh](https://ntfy.sh/) - HTTP-based pub-sub notifications.
- [Gotify](https://gotify.net/) - a simple server for sending and receiving messages.
- [Pushbullet](https://www.pushbullet.com/) - connecting different devices with cross-platform features.
- [Telegram](https://telegram.org/) - Telegram chat API.
- [Matrix-Synapse](https://github.com/element-hq/synapse) - [Matrix](https://matrix.org/), open, secure, decentralised communication.
- [Pushover](https://pushover.net/) - Simple Notifications (to your phone, wearables, desktops)
- [Discord](https://support.discord.com/hc/en-us/articles/228383668-Intro-to-Webhooks) - Discord webhooks.

Further additions are welcome - suggestions or PR!  
<sub><sup>Initiated and first contributed by [yoyoma2](https://github.com/yoyoma2).</sup></sub>  

### :date: Release notes addon to Notifications
There's a function to use a lookup-file to add release note URL's to the notification message.    
Copy the notify_templates/`urls.list` file to the script directory, it will be used automatically if it's there.   Modify it as necessary, the names of interest in the left column needs to match your container names.   
The output of the notification will look something like this:
```
Containers on hostname with updates available:
apprise-api  ->  https://github.com/linuxserver/docker-apprise-api/releases
homer  ->  https://github.com/bastienwirtz/homer/releases
nginx  ->  https://github.com/docker-library/official-images/blob/master/library/nginx
...
```
The `urls.list` file is just an example and I'd gladly see that people contribute back when they add their preferred URLs to their lists.

## :bookmark: Prometheus
Dockcheck is capable to export metrics to prometheus via the text file collector provided by the node_exporter. In order to do so the -c flag has to be specified followed by the file path that is configured in the text file collector of the node_exporter.

The following metrics are exported to prometheus

```
# HELP dockcheck_images_analyzed Docker images that have been analyzed
# TYPE dockcheck_images_analyzed gauge
dockcheck_images_analyzed 22
# HELP dockcheck_images_outdated Docker images that are outdated
# TYPE dockcheck_images_outdated gauge
dockcheck_images_outdated 7
# HELP dockcheck_images_latest Docker images that are outdated
# TYPE dockcheck_images_latest gauge
dockcheck_images_latest 14
# HELP dockcheck_images_error Docker images with analysis errors
# TYPE dockcheck_images_error gauge
dockcheck_images_error 1
# HELP dockcheck_images_analyze_timestamp_seconds Last dockercheck run time
# TYPE dockcheck_images_analyze_timestamp_seconds gauge
dockcheck_images_analyze_timestamp_seconds 1737924029
```

Once those metrics are exported they can be used to define alarms as shown below

```
- alert: dockcheck_images_outdated
  expr: sum by(instance) (dockcheck_images_outdated) > 0
  for: 15s
  labels:
    severity: warning
  annotations:
    summary: "{{ $labels.instance }} has {{ $value }} outdated docker images."
    description: "{{ $labels.instance }} has {{ $value }} outdated docker images."
- alert: dockcheck_images_error
  expr: sum by(instance) (dockcheck_images_error) > 0
  for: 15s
  labels:
    severity: warning
  annotations:
    summary: "{{ $labels.instance }} has {{ $value }} docker images having an error."
    description: "{{ $labels.instance }} has {{ $value }} docker images having an error."
- alert: dockercheck_image_last_analyze
  expr: (time() - dockcheck_images_analyze_timestamp_seconds) > (3600 * 24 * 3)
  for: 15s
  labels:
    severity: warning
  annotations:
    summary: "{{ $labels.instance }} has not updated the dockcheck statistics for more than  3 days."
    description: "{{ $labels.instance }} has not updated the dockcheck statistics for more than 3 days."
```

There is a reference Grafana dashboard in [grafana/grafana_dashboard.json](./grafana/grafana_dashboard.json).

![](./grafana/grafana_dashboard.png)

## :bookmark: Labels
Optionally add labels to compose-files. Currently these are the usable labels:
```
    labels:
      mag37.dockcheck.restart-stack: true
      mag37.dockcheck.update: true
```
- `mag37.dockcheck.restart-stack: true` works instead of the `-f` option, forcing stop+restart on the whole compose-stack (Caution: Will restart on every updated container within stack).
- `mag37.dockcheck.update: true` will when used with the `-l` option only update containers with this label and skip the rest. Will still list updates as usual.

## :roller_coaster: Workaround for non **amd64** / **arm64**
`regctl` provides binaries for amd64/arm64, to use on other architecture you could try this workaround.
Run regctl in a container wrapped in a shell script. Copied from [regclient/docs/install.md](https://github.com/regclient/regclient/blob/main/docs/install.md):

```sh
cat >regctl <<EOF
#!/bin/sh
opts=""
case "\$*" in
  "registry login"*) opts="-t";;
esac
docker container run \$opts -i --rm --net host \\
  -u "\$(id -u):\$(id -g)" -e HOME -v \$HOME:\$HOME \\
  -v /etc/docker/certs.d:/etc/docker/certs.d:ro \\
  ghcr.io/regclient/regctl:latest "\$@"
EOF
chmod 755 regctl
```
Test it with `./regctl --help` and then either add the file to the same path as *dockcheck.sh* or in your path (eg. `~/.local/bin/regctl`).

## :guardsman: Function to auth with docker hub before running
**Example** - Change names, paths, and remove cat+password flag if you rather get prompted:
```sh
function dchk {
  cat ~/pwd.txt | docker login --username YourUser --password-stdin
  ~/dockcheck.sh "$@"
}
```

## :hammer: Known issues
- No detailed error feedback (just skip + list what's skipped).
- Not respecting `--profile` options when re-creating the container.
- Not working well with containers created by **Portainer**.
- **Watchtower** might cause issues due to retagging images when checking for updates (and thereby pulling new images).

## :warning: `-r flag` disclaimer and warning
**Wont auto-update the containers, only their images. (compose is recommended)**  
`docker run` dont support using new images just by restarting a container.  
Containers need to be manually stopped, removed and created again to run on the new image.

## :wrench: Debugging
If you hit issues, you could check the output of the `extras/errorCheck.sh` script for clues. 
Another option is to run the main script with debugging in a subshell `bash -x dockcheck.sh` - if there's a particular container/image that's causing issues you can filter for just that through `bash -x dockcheck.sh nginx`.

## :scroll: License
dockcheck is created and released under the [GNU GPL v3.0](https://www.gnu.org/licenses/gpl-3.0-standalone.html) license.

## :heartpulse: Sponsorlist

- [avegy](https://github.com/avegy)

___

### :floppy_disk: The [story](https://mag37.org/posts/project_dockcheck/) behind it. 1 year in retrospect.

