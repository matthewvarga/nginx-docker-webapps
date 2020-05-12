# nginx-docker-webapps

This is the code that runs my personal webpage and other perosnal projects on an Ubuntu machine. It constsists docker images running behind a reverse-proxy (nginx) that distributes connections to the appropriate application. 

## Development

To begin development, you first must clone the repository, then pull in each submodule. To do so, after cloing the base repo, run `git submodule init` followed by `git submodule update --remote`. Any time a submodule is updated, you will need to run the second command to pull in the latest update.

If you work on a submodule, add and commit the files as usual, and push them to the server with `git push origin HEAD:branch_name` from its repective directory.

If you need to add another submodule, run `git submodule add <git_repo_url>`.

**Note:** more info on working with git submodules can be found [here](https://git-scm.com/book/en/v2/Git-Tools-Submodules).

## Deployment

This application requries only Docker and Docker-Compose to be installed on the production machine. Every submodule should include a Dockerfile to easily deploy in any environment. However, the ssl ciphers make use of ECDH, thus requiring for us to specify the DH paramaters for the DHE ciphers. To do so, you must create the directory `dhparam` from the root project directory, and then run the following to generte the cipher params:

```
sudo openssl dhparam -out PROJECT_REPO/dhparam/dhparam-2048.pem 2048
```

where PROJECT_REPO is the location of the project repository.


Next, you can simply run `docker-compose build` to build the apllication, followed by `docker-compose up -d` to start it.

To stop it, run `dock-compose down`.

Finally, if you need to clear the build files, run `docker system prune -a`.

## ssl_renew.sh

This file is a bash script that will renew the ssl certificates. Since they are only valid for 90 days, it is convenient to setup a cronjob to check if they are available for renewal, and if so, renew them. Below are the steps for setting up the cronjob, it runs the script every day at 12 (noon) to check wether or not the certs need renewal.

First, start by making the script executable with: `chmod +x ssl_renew.sh`. 

Next, open the root crontab file to speccify we would like to run the script at a specific interval: `sudo crontab -e`.

Finally, add the following line to the bottom of the file:
`0 12 * * * /home/matthew/nginx-docker-webapps/ssl_renew.sh >> /var/log/cron.log 2>&1`.

Now you do not need to worry about renewing the certs.

**NOTE:** to do a quick test run, replace the above line with
`*/5 * * * * /home/matthew/nginx-docker-webapps/ssl_renew.sh >> /var/log/cron.log 2>&1` instead. this runs every 5 minutes opposed once a day. Next, add the `--dry run` flag to the 7th line in ssl_renew.sh to indicate we are running a test. Then, after 5 minutes have surpassed, check `tail -f /var/log/cron.log` to ensure a succesful renewal took place. Once this is done, revert the changes back to align with the steps aforementioned.

## Troubleshooting

If you encounter issues where docker containers are not running properly, you can use `docker-compose logs` to view the error logs.