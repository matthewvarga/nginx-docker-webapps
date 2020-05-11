# nginx-docker-webapps

This is the code that runs my personal webpage and other perosnal projects on an Ubuntu machine. It constsists docker images running behind a reverse-proxy (nginx) that distributes connections to the appropriate application. 

## Development

To begin development, you first must clone the repository, then pull in each submodule. To do so, after cloing the base repo, run `git submodule init` followed by `git submodule update --remote`. Any time a submodule is updated, you will need to run the second command to pull in the latest update.

If you work on a submodule, add and commit the files as usual, and push them to the server with `git push origin HEAD:branch_name` from its repective directory.

If you need to add another submodule, run `git submodule add <git_repo_url>`.

**Note:** more info on working with git submodules can be found [here](https://git-scm.com/book/en/v2/Git-Tools-Submodules).

## Deployment

This application requries only Docker and Docker-Compose to be installed on the production machine. Every submodule should include a Dockerfile to easily deploy in any environment.

To build the docker images, first run `docker-compose build`. Then, once the build process is finished, run `docker-compose up -d` to start the application.

To stop it, run `dock-compose down`.

Finally, if you need to clear the build files, run `docker system prune -a`.
