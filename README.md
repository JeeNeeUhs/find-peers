# Find peers --> [find-peers.jeeneeuhs.dev](https://find-peers.jeeneeuhs.dev)

This website is meant to help students to find peers that are working on the same project

## Developing
## Setup & configuration
- Create a Oauth application on [intra](https://profile.intra.42.fr/oauth/applications)
- Copy the file `./env/.env-example` to `./env/.env` and fill out the (secret) data

Also see `./src/env.ts` for more configuration

## Changing listed projects
- The projects shown on the front page are listed in `./env/projectIDs.json`. Should the curriculum change, you can edit that file. Remember to restart the server and wait for the server to pull all the data from the intra api.
- A list of all the projects and their corresponding ID in the 42 network (as of march 2022) can be found in `./env/allProjectIDs.json`

## Updating the secrets / API tokens
```shell
cd find-peers
vim env/.env
# make changes
docker compose down
docker compose up -d

# To get logs
docker logs --tail 10000 -f find-peers
```

## Monitoring
At the (unauthenticated) route `/status/pull` you can see a summary of the pull status of every campus
It contains the key `hoursAgo` for every campus, which is the amount of hours since the last successful pull (syncing the 42 DB of the users' completed projects) of that campus
This value should not be higher than 2 * the pull timeout (currently 24 hours)

## Configuration files
| File path                                    | Description                                                                           | Managed by server |
|----------------------------------------------|---------------------------------------------------------------------------------------|-------------------|
| `./env/projectIDs.json`                      | List of all the projects and their corresponding ID to be displayed on the front page | no                |
| `./env/allProjectIDs.json`                   | List of all projects in the 42 network (as of march 2022)                             | no                |
| `./env/.env-example`                         | Example file for api tokens, rename to `.env` to activate                             | no                |
| `./env/campusIDs.json`                       | List of running campuses and their corresponding ID that are fetched from the 42 API  | no                |
| `./env/allCampusIDs.json`                    | List of all campuses and their corresponding ID that are fetched from the 42 API      | no                |
| `./database/`                                | All database files, mount this when running in a docker container                     | yes               |
| `./database/sessions/`                       | All session files currently active                                                    | yes               |
| `./database/users.json`                      | Userdata associated with session                                                      | yes               |
| `./database/<campus_name>/lastpull.txt`      | Unix timestamp when the project users of that campus were last successfully updated   | yes               |
| `./database/<campus_name>/projectUsers.json` | Status of users for each project                                                      | yes               |

## Running
The 'database' of this project is a folder called 'database' at the root of the project.

# Docker and Docker-compose
This is in production use this commands
```shell
git clone https://github.com/jeeneeuhs/find-peers.git
cd find-peers
docker compose up -d

# To get logs
docker logs --tail 10000 -f find-peers
```
# Change campuses
if you want to change the used campuses you will need to rebuild
```shell
git clone https://github.com/jeeneeuhs/find-peers.git
cd find-peers
```
add or remove any campus ids you want 
```shell
#all campus ids
cat env/allCampusIDs.json

#add or delete the campuses you will use
nano env/campusIDs.json
```
build your own docker
```shell
docker build -t myfindpeersbuild .
```

change docker-compose image
```yaml
version: "3.7"

services:
  find-peers:
    container_name: find-peers
    restart: unless-stopped
    image: myfindpeersbuild  # change here like this
    volumes:
      - $HOME/find-peers/database:/app/database
    env_file: ./env/.env
    environment:
      - PORT=8080
    ports:
      - 80:8080

```
run docker
```shell
docker compose up -d
```