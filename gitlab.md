# gitlab

## Create new repository

- click on `New Project` in the gitlabe group of your choice (e.g. your gitlab home). Can be also changed later.

- Create Blank Project

- Uncheck `Initialize repository with a README` if you already have create a repository locally.

  - Now a new repository has been created with a receip to fill it with life. Under "Push an existing folder" is described which commands might be good to fill the repo with life. Usually, one only needs

        ```bash
        git remote add origin ssh://git@my-server.de:1022/path/to/my-project.git
        git push -u origin main
        ```
