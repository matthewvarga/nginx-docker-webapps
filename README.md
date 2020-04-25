# nginx-docker-webapps

## submodules

To **add** a new submodule, use: `git submodule add <git_repository_url>`

https://git-scm.com/book/en/v2/Git-Tools-Submodules

## Development

Method 1.

1. clone the project: `git clone ...`

2. initialize submodules: `git submodule init`

3. fetch submodules data: `git submodule update`

Method 2.

1. clone the project with submodules: `git clone ... --recurse-submodules`


--

pull submodule changes from remote repo: `git submodule update --remote`


-- To push submodule changes, enter the submodule, add and commit the files, the run `git push origin HEAD:branch_name`