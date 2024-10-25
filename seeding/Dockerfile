FROM mongo:8.0-rc

COPY seeding.json /seeding.json

CMD mongoimport --host mongo --db tmdb_movies --collection movies --type json --file /seeding.json --jsonArray --username admin --password password --authenticationDatabase admin