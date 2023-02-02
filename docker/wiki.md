# Installation
## MacOS
Choose your correct architecture
https://docs.docker.com/desktop/install/mac-install/

## Linux
Easy install script

`curl -fsSL https://get.docker.com -o get-docker.sh`

`sh get-docker.sh`

# Start
Inside the root directory of the project run the following command
`docker compose build app`
after the command has finished run
`docker compose up -d`

# Running php commands
`./exec php -v`
