Check out the main branch of our GitHub repository: 
```bash
git clone https://github.com/cereal-lab/EvoPIE.git
```

Edit the docker-compose.yml file to update the volumes for "web". Put the absolute path to the folder containing the database file where we have "REPLACE_ME" below: 
```bash
version: '2.0'

services:
  web:
    build: ./evopie
    volumes:
      - /REPLACE_ME:/app/data
    environment:
      - EVOPIE_DATABASE_URI=sqlite:////app/data/db.sqlite
    expose:
      - 5000
    env_file:
      - ./evopie/.env.dev
    restart: always
  nginx:
    build: ./nginx
    ports:
      - "5000:5000"
    depends_on:
      - web
    restart: always
    volumes:
      - /etc/letsencrypt:/etc/nginx/certs
```

Build the docker containers and run them:
```bash
docker compose up --build -d
```
(Note the space since docker-compose is now deprecated and replaced by the command compose in docker)
