# cbioportal-docker

Download and install Docker from www.docker.com.

[Notes for non-Linux systems](notes-for-non-linux.md)


#### Step 1 - Setup network
Create a network in order for the cBioPortal container and mysql database to communicate.
```
docker network create cbio-net
```

#### Step 2 - Run mysql with seed database
Download the seed database from https://github.com/thehyve/cbioportal/blob/new_seed_db/docs/Downloads.md#seed-database

This command imports the seed database file into a database stored in
`/<path_to_save_mysql_db>/db_files/` (:warning: this should be an absolute path in command below), before starting the MySQL server. 

This process takes about 45 minutes. For much faster loading, we can choose to not load the PDB data, by removing the line that loads cbioportal-seed_only-pdb.sql.gz.

```
docker run -d --name "cbioDB" \
  --restart=always \
  --net=cbio-net \
  -p 8306:3306 \
  -e MYSQL_ROOT_PASSWORD=P@ssword1 \
  -e MYSQL_USER=cbio \
  -e MYSQL_PASSWORD=P@ssword1 \
  -e MYSQL_DATABASE=cbioportal \
  -v /<path_to_save_mysql_db>/db_files/:/var/lib/mysql/ \
  -v /<path_to_seed_database>/cbioportal-seed_no-pdb_hg19.sql.gz:/docker-entrypoint-initdb.d/seed_part1.sql.gz:ro \
  -v /<path_to_seed_database>/cbioportal-seed_only-pdb.sql.gz:/docker-entrypoint-initdb.d/seed_part2.sql.gz:ro \
  mysql
```

#### Step 3 - Build the Docker image containing cBioPortal
Checkout the repository, enter the directory and run build the image.

```
cd cbioportal-docker
git clone https://github.com/thehyve/cbioportal-docker.git
docker build -t custom/cbioportal .
```

Alternatively, if you do not wish to change anything in the Dockerfile or the properties, you can run:

```
docker build -t cbioportal https://github.com/thehyve/cbioportal-docker.git
```

#### Step 4 - Run the cBioPortal container
```
docker run -d --name="cbioportal" \
  --restart=always \
  --net=cbio-net \
  -p 8081:8080 \
  custom/cbioportal
```

cBioPortal can now be reached at http://localhost:8081/cbioportal/

Activity of Docker containers can be seen with:
```
docker ps -a
```

## Uninstalling cBioPortal
First we stop the Docker containers.
```
docker stop cbioDB
docker stop cbioportal
```

Then we remove the Docker containers.
```
docker rm cbioDB
docker rm cbioportal
```

Cached Docker images can be seen with:
```
docker images
```

Finally we remove the cached Docker images.
```
docker rmi mysql
docker rmi custom/cbioportal
```

## Data Loading & More commands

For more uses of the cBioPortal image, see [example_commands.md](example_commands.md)
