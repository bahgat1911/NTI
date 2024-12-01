# Use the official Node.js image as a base
FROM node:16

# Set the working directory inside the container
WORKDIR /app

# Copy package.json and package-lock.json to install dependencies
COPY package*.json ./

# Install the app dependencies
RUN npm install

# Copy the rest of the app source code
COPY . .

# Download and install the 'wait-for-it' script to ensure MongoDB is ready before starting
RUN curl -sS https://raw.githubusercontent.com/vishnubob/wait-for-it/master/wait-for-it.sh -o /wait-for-it.sh
RUN chmod +x /wait-for-it.sh

# Wait for MongoDB to be ready
CMD /wait-for-it.sh database:27017 -- npm start
# Expose the port the app will run on
EXPOSE 3001

