####### Create a container first#########
# 1. create a volume to be mounted
sudo docker volume pg_data
# 2. Create the container with volume mounted
sudo docker run --name=postgis -d -e POSTGRES_USER=robin -e POSTGRES_PASS=****** -e POSTGRES_DBNAME=robin -e ALLOW_IP_RANGE=0.0.0.0/0 -p 5432:5432 -v pg_data:/var/lib/postgresql --restart=always kartoza/postgis:latest

####### Download the database dump from internet#######
wget http://minus34.com/opendata/psma-201808/gnaf-201808.dmp
wget http://minus34.com/opendata/psma-201808/admin-bdys-201808.dmp
wget http://minus34.com/opendata/census-2016/census_2016_data.dmp
http://minus34.com/opendata/census-2016/census_2016_bdys.dmp
http://minus34.com/opendata/census-2016/census_2016_web.dmp

####### Restore database from dump file#######
# 1. ABS Census database
pg_restore --host "127.0.0.1" --port "5432" --username "robin" --password --dbname "robin" --verbose "./census_2016_data.dmp"
# 2. ABS Boundry database
pg_restore --host "127.0.0.1" --port "5432" --username "robin" --password --dbname "robin" --verbose "./census_2016_bdys.dmp"
# 3. ABS Census Web database
pg_restore --host "127.0.0.1" --port "5432" --username "robin" --password --dbname "robin" --verbose "./census_2016_web.dmp" 
# 4. Admiistrative Boundry database
pg_restore --host "127.0.0.1" --port "5432" --username "robin" --password --dbname "robin" --verbose "./admin-bodys-201808.dmp"
# 5. GNAF database
pg_restore --host "127.0.0.1" --port "5432" --username "robin" --password --dbname "robin" --verbose "./gnaf-201808.dmp"

####### Create Read_Only_User for database access
# 1.Create read_only_user in postgresql database (db:dataraft)
CREATE ROLE Read_Only_User WITH LOGIN PASSWORD 'Test1234' 
NOSUPERUSER INHERIT NOCREATEDB NOCREATEROLE NOREPLICATION VALID UNTIL 'infinity';

GRANT USAGE ON SCHEMA admin_bdys_201808 TO Read_Only_User;
GRANT USAGE ON SCHEMA census_2016_bdys TO Read_Only_User;
GRANT USAGE ON SCHEMA census_2016_data TO Read_Only_User;
GRANT USAGE ON SCHEMA census_2016_web TO Read_Only_User;
GRANT USAGE ON SCHEMA gnaf_201808 TO Read_Only_User;
GRANT SELECT ON ALL TABLES IN SCHEMA admin_bdys_201808 TO Read_Only_User;
GRANT SELECT ON ALL TABLES IN SCHEMA census_2016_bdys TO Read_Only_User;
GRANT SELECT ON ALL TABLES IN SCHEMA census_2016_data TO Read_Only_User;
GRANT SELECT ON ALL TABLES IN SCHEMA census_2016_web TO Read_Only_User;
GRANT SELECT ON ALL TABLES IN SCHEMA gnaf_201808 TO Read_Only_User;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA admin_bdys_201808 TO Read_Only_User;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA census_2016_bdys TO Read_Only_User;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA census_2016_data TO Read_Only_User;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA census_2016_web TO Read_Only_User;
GRANT SELECT ON ALL SEQUENCES IN SCHEMA gnaf_201808 TO Read_Only_User;

# 2.Change User Password
ALTER USER Read_Only_User WITH PASSWORD 'dataraft2018';

##### Create pgAdmin4 database management tool######
sudo docker pull dpage/pgadmin4
docker run -p 80:80 -e "PGADMIN_DEFAULT_EMAIL=abc@abc.com.au" -e "PGADMIN_DEFAULT_PASSWORD=******" -d dpage/pgadmin4

