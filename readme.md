# pre: setup a PostGIS DB on your localhost with name bagv2 and postgres postgres credentials

# Build Docker Image - Optional, better or get the Image from DockerHub

# See https://hub.docker.com/repository/docker/nlextract/nlextract/

cd ../..
./build-docker.sh
cd -

# Get the Docker Image from Docker Hub

docker pull nlextract/nlextract:latest

# Case 1: Run without args: default small internal testdata set to test basic setup

# etl.sh can accept Stetl-based -a options

docker run --rm --name nlextract nlextract/nlextract:latest bagv2/etl/etl.sh
echo "SELECT \* FROM test.verblijfsobject" | psql bagv2
echo "\d+ test.verblijfsobject" | psql bagv2;

# Case 2: single Municipality

# Create local bag dir

mkdir bag

# Download a BAG XML file from https://extracten.bag.kadaster.nl/lvbag/extracten

# For example Doesburg (code 0221 march 2021): BAGGEM0221L-15032021.zip

curl -o bag/bag.zip https://extracten.bag.kadaster.nl/lvbag/extracten/Gemeente%20LVC/0221/BAGGEM0221L-15032021.zip

# Single Municipality (Gemeente)

# Override default PG schema and default input file via Docker Volume mapping and Stetl -a options:

docker run --name nlextract --rm -v $(pwd)/bag:/bag nlextract/nlextract:latest bagv2/etl/etl.sh -a schema=doesburg -a bag_input_file=/bag/bag.zip

# Test result

echo "select count(gid) from doesburg.pand" | psql bagv2

# count

# -------

# 13390

# (1 row)

# Case 3: Entire Netherlands (march 2021)

curl -o bag/bag.zip https://www.kadaster.nl/-/kosteloze-download-bag-2.0-extract
docker run --name nlextract --rm -v $(pwd)/bag:/bag nlextract/nlextract:latest bagv2/etl/etl.sh -a bag_input_file=/bag/bag.zip

# Stopping a process

docker stop nlextract
