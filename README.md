# Docker Template: React on Rails with PostgreSQL

## What Did I Just Do? 
This project serves as a template that will allow you to toss your React on Rails (Ruby on Rails backend with React fronend) application into a folder, configure a few settings, then quickly have a Dockerized app with it's own contained environment! From there, the app can be deployed on a VPS (Virtual Private Server) with almost no environment configuration. Using Docker also makes it easier to manage multiple apps running on the same VPS, which makes it very cost-effective when you have more than a few apps deployed. This template aims to make Docker as easy to setup as it is to use!

## Installation Instructions
### _5-second automatic version_
1. Run 

### _5-minute manual version_
1. Copy and paste your React on Rails application inside `ror/` so that `ror/` is the root directory of your RoR app.
2. In `.docker/` copy the `.env.example` file into the same directory and rename it `.env`. You do not need to change any variables here in your local repository, but you will need to configure this file on your server repo.
3. Copy and paste the contents of `.docker/example.database.yml` into `ror/config/database.yml` and replace the contents inside, being careful of any database configurations you have already set for your project (if any).
4. Open `docker-compose.yml` file in this project's root directory and change the first port number only, to specify which port you want your app to run on. For example change it to `"4000:3000"` if you want to run the app on port 4000. Do not change the second number.
5. Open your terminal in the root directory and enter `bin/build-frontend`.
6. **Your app is now Dockerized!!** Visit the 'Operation Instructions' section below to read about how to get the app on your server and run Docker Compose, if you have never used it before.

_Note: If you are using React Router Dom with your app, please visit that section below, as there are a few extra steps for deployment. The steps are not Docker specific. It would be the same steps, regardless of the deployment method._

## Operation Instructions
1. You can now upload your app to GitHub and then pull it down onto your server. Once on your server, go to `.docker/` directory in your server project repo and repeat step 2 above. This time, choose a secure password and set the correct variable to your chosen password.
2. Now copy the master key from your local repo in `ror/config/master.key` and paste that key either into `ror/config/master.key` on your server repo, _or_ uncomment the `RAILS_MASTER_KEY` variable in the `.docker/.env` and paste the key in that line in on your server repo.
3. **You can run your app now!!** Make sure Docker and Docker Compose are installed on your server, then **run `sudo docker compose up --build -d`** from your project's root directory to build the image on your server and run your app. To see your app, **visit `http://<YOUR_SERVERS_PUBLIC_IP>:<PORT>`**. To take down your app, run `sudo docker compose down`. If your app is running and you've made changes to it and want to update the app on the server, I've included a useful script that will automatically run `git pull`, bring your app offline, rebuild the static frontend files, rebuild the Docker image, and bring it back online, all in one command! **Simply run `bin/update-app`** from the root directory.

## Where To Go Next
If you are new to VPS server deployment and are wondering how you get your domain name connected, and more, here are some next steps to look into:
1. Buy a domain from Name Cheap or Google Domains. Usually this is around $10-$12 per year.
2. Go into the DNS settings and create an A record that points to your server's public IP address (the IP should be provided by your server platform provider. Usually it's in your web portal after you login.).
3. Install a routing software like Nginx to forward requests to the appropriate port that your app is running on.
4. Install Let's Encrypt on your server to install an SSL Certificate so your server can accept HTTPS and so your browser sees your site as safe.

---

## 5-Hour Explanation
This is an explanation of the steps taken above and is more for your education than it is necessary to read.
### _Going over the 5-minute manual version_
1. The `ror/` directory stands for Ruby on Rails or React on Rails. Your React app should then be inside the `client/` folder inside of your ruby app directory.
2. The .docker/.env file contains environment variables that get set in your Docker Container. These variables are loaded into both the database's container and the app's container. By sharing the same environment variables, they can share useful information, like the database's password, active user and the Rails environment. The master key is also stored here. This master key is used for encrypting data and should not be public, which is why git ignores it and also why you must manually load the master key onto the server's repo, even though your local repo already has it. Rails is able to view the master key when it is stored in either the `.docker/.env` file as an environment variable, or in it's own `master.key` file in the `ror/config/` directory. Either option works for Rails, so how you store the key is entirely up to you. If you need to generate a new one, use `rails credentials:edit` command. That will generate a new `master.key` file, but will also recreate some other important codes, so be sure that you update all other repos after issuing that command. The other files it updates are not ignored by git (which is a good thing) so you can use normal git commands like `git push` and `git pull` to update your other repos.
3. The `ror/config/database.yml` file is a configuration file for you database. In order for it to work with Docker, there are a few modifications that need to be made: A host needs to be specified and a username and password needs to be required and passed in, for security. Anything enclosed in `<%= %>` is actually ruby code. The username and password use `ENV["VAR"]` instead of `ENV.fetch("VAR")` to fetch environment variables because the first option returns `nil` if the variable is not found, while the second one throws an error. By using the first option for username and password, it allows the username and password to be optional, since those environment variables will be established in the Docker Container (via the `.docker/.env` file), and are not likely established on your computer. Essentially, a password is not required when you are running it locally, but is required when it is running on your server through Docker. Similar story with the host key. If the `POSTGRES_PASSWORD` environment variable is not found, the program assumes that you are on your local machine and does not need to specify a host. But when the database and app are running through separate containers, a host name is needed.
4. The `docker-compose.yml` file is where `docker compose` gets it's instructions to build and run your app. Probably the most important part of this file is the port number. To adjust the port number that the app is running on, you must change the first of the two numbers. So entering `"4000:3000"` would run the app on port 4000. The reason the second number can't be changed is because that number is specifying the internal port number. This is the port number that Rails is technically running on inside the Docker Container, but then Docker forwards this internal port to an external port, which can be accessed outside the container. This external port number is the first number (port 4000 in my example case above). Inside this file, you can also see things like the `env_file` key, which is where the `.env` file is referenced, which contains the environment variables for the container. Additionally, the `dockerfile` key-value pair points to the Dockerfile, which contains instructions on how to build the app. Feel free to open that up and take a look if you are curious. 
5. There are more custom scripts located in the `bin/` folder. Feel free to open them up with a text editor to see what's inside! Keep in mind, these are bash scripts, not ruby scripts.
6. Once the app is dockerized, it can be taken to the server to be deployed.

## ***FREE*** Knowledge! (A.K.A. tips and FAQs)
### _Using React Router Dom?_
This forwards all unknown `GET` requests to `fallback#index`, which returns your React app. Without these steps, refreshing the page while on any page except the home page will result in a 404 error. This error is not Docker-specific, but instead is React on Rails-specific when using React Router Dom.
1. Create a controller called `fallback_controller.rb` in `ror/app/controllers/` and paste this code in:
```
class FallbackController < ActionController::Base
  def index
    render file: 'public/index.html'
  end
end
```
2. Open the routes file, found in `ror/config/routes.rb`, and add this line - `get "*path", to: "fallback#index", constraints: ->(req) { !req.xhr? && req.format.html? }` - to the bottom of the main code block so it looks something like this:
```
Rails.application.routes.draw do
  ...
  get "*path", to: "fallback#index", constraints: ->(req) { !req.xhr? && req.format.html? }
end
```

### _Docker tips & commands_
#### _TIPS_
1. Docker commands should always be run with `sudo` because the Docker daemon is being run with admin priviledges. There is a way to get around it, but to me it just seems easier to use `sudo`. Feel free to Google around if that's the direction you want to go.
2. Some installations of Docker Compose come with an alias, allowing you to use the `docker-compose` command instead of `docker compose`. Others do not, so I use `docker compose` in this instructional set, as well as in my custom bash scripts.
3. A Docker Image is like a screenshot of your app with it's environment included. The image even includes it's own Linux distro and file system. However, since it is a screenshot, the original cannot be modified. If you make a change, you must take a new screenshot.
4. A Docker Container runs a Docker Image. It allows code to be stored in memory so your app can be dynamic. It is similar to a virtual machine, but instead of the environment being stored in the VM, it is stored in the image. Containers are much more performant than VMs.
5. A Docker Volume is a way of syncing data between a specified folder on your container with a folder on your local machine. The reason these are necessary is because, remember, data cannot be persisted to Docker Images, since images cannot be modified. Since this is the case, the Docker Container stores the data in memory and syncs it with a folder on the local machine, so when the docker is shut off, the data can be accessed next time a connected container is run. The best use for these volumes is to persist data in a database. In this project, the database in the Docker Container is syncing with our project folder `.docker/volumes/database/`. If this data is ever needed to be accessed in the future, that is where you can find it.
6. A Docker Network ties two or more Docker Containers together and allows them to interact with each other. In the example app included with this project, the PostgreSQL database is running in a container, and the RoR app is running in a seperate container. They are able to communicate with each other through the Docker Network.
7. Docker Compose is a solution for allowing you to configure Docker settings in a YAML file, instead of typing those settings in the command every time you run a container. For example, after specifying the port, environment variables, network, and more, the command to run my container might look like this: `sudo docker -p 3000:3000 -e POSTGRES_USER=rails -e POSTGRES_PASSWORD=1234 -e RAILS_ENVIRONMENT=production --network host ...` etc, etc. By placing my settings in a YAML file, Docker Compose can read the file and apply those settings automatically.
#### _COMMANDS_
_Note: When using `docker compose` commands, be sure you are in the root directory of the project you want to manipulate. Docker Compose uses the `docker-compose.yml` file in the root directory to know which containers to apply your commands to._
1. `sudo docker compose build` will build a docker image with your app in it and save it to a mysterious directory on your computer. Images _do not ever change_, so anytime you update your app, you must build a new image to replace the old. This command can be used for that. Everytime you build an image, it replaces the old automatically.
2. `sudo docker compose up` will take the image built from the directory you are in and run it on the port specified in the `docker-compose.yml` file! If you add the `--build` flag, it will build a new image first, then run that image. It essentailly functions like `sudo docker compose build` immediately followed by `sudo docker compose up`. If you add the `-d` flag, it runs your app in the background, so you don't need to stare at a terminal the whole time your app is up. My favorite command to use when updating my app on my server is `sudo docker compose up --build -d` because it builds my app, then runs it in the background.
3. `sudo docker compose down` will take your app offline.
4. `sudo docker ps` will show you all containers that are online and listening on one of your ports. `sudo docker ps -a` will show you all containers that are online _or_ offline.
5. `sudo docker images` will show all images saved on your system.
6. `sudo docker stop <CONTAINER_ID>` takes a container offline, using the container's ID.
7. `sudo docker rm <CONTAINER_ID>` deletes a container from your local storage. If you login to root using `su` and then typing your password, you can use `docker rm $(docker ps -aq)` to delete all containers on your system. Notice `sudo` is not used. That is because as root, you don't need `sudo` to elevate your privileges, since root already has highest privileges.
8. `sudo docker rmi <IMAGE_ID>` deletes an image from your local storage. _Before deleting images, make sure a container is not reliant on that image._ If you login to root using `su` and then typing your password, you can use `docker rmi $(docker images -aq)` to delete all images on your system. Notice `sudo` is not used here either.