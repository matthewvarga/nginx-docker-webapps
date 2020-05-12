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

## Troubleshooting

If you encounter issues where docker containers are not running properly, you can use `docker-compose logs` to view the error logs.