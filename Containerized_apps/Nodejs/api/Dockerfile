# Fetching the latest node image on apline linux
FROM node:17-alpine

# Globally install nodemon 
RUN npm install -g nodemon

# Setting up the work directory
WORKDIR /app

# Copying all the files in our project
COPY . .

# Installing dependencies
RUN npm install

# Port to be exposed
EXPOSE 4000

# Starting our application
CMD ["npm", "run", "dev"]