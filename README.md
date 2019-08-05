# Minecraft Bedrock Server
Minecraft Bedrock edition for Docker ( also Synology )

## Usage

#### Prerequisites
1. Working docker installation

### For Synology
1. From the Docker application, navigate to Registry, and find "glemsom/bedrock-server" image (earch for glemsom)
2. Right-click, and select Download this image
3. Select the desired version from tags (NOTE: Latest is development tag, and might break at random times)
4. Navigate to Image, and Launch an instance of "glemsom/bedrock-server"
5. Choose "Advanced Settings"
6. OPTIONAL: Select "Enable auto-restart"
7. Under the Volume tab, choose "Add Folder" - and find (or create) a folder for storing Bedrock configuration and world files. Mount the folder as `/bedrock-server/data`
8. Under Port Settings, change "Local Port" from "Auto" to 19132
9. OPTIONAL: Console access to Bedrock
Navigate to Docker -> Container -> Details -> Terminal


### For regular docker
1. Prepare the persistent volumes:
    1. Create a volume for the Bedrock:<br/>
        `docker volume create --name "bedrock-data"`
2. Create the Docker container:
    ```bash
    docker create --name=minecraft \
        -v "bedrock-data:/bedrock-server/data" \
        -p 19132:19132/udp \
        --restart=unless-stopped \
        glemsom/bedrock-server:latest
    ```
    
3. Start the server:
    ```bash
     docker run -v "bedrock:/bedrock-server/data" \
     -p 19132:19132/udp glemsom/bedrock-server:latest`

## Commands
There are various commands that can be used in the console.
To access the console, you need to attach to the container with the following command:
```
docker attach <container-id>
```
To leave the console without exiting the container, use `Ctrl`+`p` + `Ctrl`+`q`.

Here are the commands:

| Command syntax | Description |
| -------------- | ----------- |
| kick {`player name` or `xuid`} {`reason`} | Immediately kicks a player. The reason will be shown on the kicked players screen. |
| stop | Shuts down the server gracefully. |
| save {`hold` or `resume` or `query`} | Used to make atomic backups while the server is running. See the backup section for more information. |
| whitelist {`on` or `off` or `list` or `reload`} | `on` and `off` turns the whitelist on and off. Note that this does not change the value in the `server.properties` file!<br />`list` prints the current whitelist used by the server<br />`reload` makes the server reload the whitelist from the file.
| whitelist {`add` or `remove`} {`name`} | Adds or removes a player from the whitelist file. The name parameter should be the Xbox Gamertag of the player you want to add or remove. You don't need to specify a XUID here, it will be resolved the first time the player connects. |
| permissions {`list` or `reload`} | `list` prints the current used permissions list.<br />`reload` makes the server reload the operator list from the permissions file. |
| op {`player name`} | Promote a player to `operator`. This will also persist in `permissions.json` if the player is authenticated to XBL. If `permissions.json` is missing it will be created. If the player is not connected to XBL, the player is promoted for the current server session and it will not be persisted on disk. Default server permission level will be assigned to the player after a server restart. |
| deop {`player name`} | Demote a player to `member`. This will also persist in `permissions.json` if the player is authenticated to XBL. If `permissions.json` is missing it will be created. |
| changesetting {`setting`} {`value`} | Changes a server setting without having to restart the server. Currently only two settings are supported to be changed, `allow-cheats` (`true` or `false`) and `difficulty` (0, `peaceful`, 1, `easy`, 2, `normal`, 3 or `hard`). They do not modify the value that's specified in `server.properties`. |

## Backups
The server supports taking backups of the world files while the server is running. It's not particularly friendly for taking manual backups, but works better when automated. The backup (from the servers perspective) consists of three commands:

| Command | Description |
| ------- | ----------- |
| save hold | This will ask the server to prepare for a backup. It’s asynchronous and will return immediately. |
| save query | After calling `save hold` you should call this command repeatedly to see if the preparation has finished. When it returns a success it will return a file list (with lengths for each file) of the files you need to copy. The server will not pause while this is happening, so some files can be modified while the backup is taking place. As long as you only copy the files in the given file list and truncate the copied files to the specified lengths, then the backup should be valid. |
| save resume | When you’re finished with copying the files you should call this to tell the server that it’s okay to remove old files again. |
