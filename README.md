# Problem statement - Build a Dockerfile for deploying a simple Ruby on Rails application with PostgreSQL DB enabled. Application and DB should run on different containers.

  Source code - evans22j /Budget-App - GitHub Repo

# Step 1: Install Docker and Docker Compose

First, ensure that Docker and Docker Compose are installed on your system.
Docker: You can install Docker from the official Docker website.

https://docs.docker.com/engine/install/ubuntu

Docker Compose: You can install Docker Compose from the official Docker Compose website.-

  https://docs.docker.com/engine/install/ubuntu/#install-using-the-repository
  
Commands for installing docker engine & docker-compose :–

# 1 Update the apt package index and install packages to allow apt to use a repository over HTTPS:

$ sudo apt-get update

$ sudo apt-get install \
apt-transport-https \
ca-certificates \
curl \
software-properties-common

# 2 Add Docker’s official GPG key :-

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add –

# 3 Set up the stable repository:

$ sudo add-apt-repository \ "deb [arch=amd64] https://download.docker.com/linux/ubuntu \ $(lsb_release -cs) \ stable"

# 4 Install Docker Engine:

$ sudo apt-get update

$ sudo apt-get install docker-ce docker-ce-cli containerd.io -y

# 5 Start Docker

$ sudo systemctl start docker

$ sudo systemctl enable docker

$ sudo systemctl status docker

step 1: Install Docker Compose:

$ sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

$ sudo chmod +x /usr/local/bin/docker-compose

Step 2: Clone the GitHub Repository

$ git clone https://github.com/evans22j/Budget-App.git

$ cd Budget-App

Step 3: Create the Dockerfile
In the root directory of the cloned repository, create a Dockerfile

$ cd Budget-App

$ sudo cat > Dockerfile

FROM ruby:3.1.2

ENV RAILS_ENV=development
ENV RACK_ENV=development

RUN apt-get update -qq && apt-get install -y nodejs postgresql-client

WORKDIR /app

COPY Gemfile /app/Gemfile
COPY Gemfile.lock /app/Gemfile.lock

RUN bundle install

COPY . /app

RUN bundle exec rake assets:precompile

EXPOSE 3000

CMD ["rails", "server", "-b", "0.0.0.0"] 

# Step 4: Create the Docker Compose File
In the same root directory, create a docker-compose.yml

$ cd Budget-App

$ sudo cat > docker-compose.yml

version: '3.8'

services:

  db:
  
    image: postgres:13
    
    volumes:
    
      - postgres_data:/var/lib/postgresql/data
    environment:
    
      POSTGRES_DB: budget_app_development
      
      POSTGRES_USER: postgres
      
      POSTGRES_PASSWORD: password

  web:
  
    build: .
    
    command: bash -c "rm -f tmp/pids/server.pid && bundle exec rails s -b '0.0.0.0'"
    
    volumes:
    
      - .:/app
      
    ports:
    
      - "3000:3000"
      
    depends_on:
    
      - db
    environment:
    
        DATABASE_URL: postgres://postgres:password@db:5432/budget_app_development


volumes:

  postgres_data:


# Step 5: Update database.yml

Update the config/database.yml

$ cd Budget-App/config

$ ls

$ rm -rvf database.yml


create new database.yml

$ sudo vim database.yml

# For Rails application to match the PostgreSQL service configuration in the docker-compose.yml file:

$ sudo cat > database.yml
[
default: &default

  adapter: postgresql
  
  encoding: unicode
  
  host: db
  
  username: postgres
  
  password: password
  
  pool: 5


development:

  <<: *default

  database: budget_app_development


test:

  <<: *default
  
  database: budget_app_test


production:

  <<: *default
  
  database: budget_app_production
  
  username: budget_app
  
  password: <%= ENV['BUDGET_APP_DATABASE_PASSWORD'] %>
]

# Step 6: Build and Run the Containers

Build and run the containers using Docker Compose:

$ Sudo docker-compose up –build

# Step 7: Open dublicate of cloudshell

# Step 8: Set Up the Database

Once the containers are up and running, you need to create and
migrate the database. Open a new terminal and run the following

$ docker-compose run web rake db:create db:migrate

# Step 8: Access the Application

http://localhost:3000

Images of successfully deployment of Ruby on Rails application with
PostgreSQL DB using docker
