# Use the official Node.js image
FROM node:16 AS builder

# Set the working directory
WORKDIR /app

# Copy package.json and package-lock.json
COPY package*.json ./

# Install dependencies
RUN npm install

# Copy the .env.production file
COPY .env.production ./

# Copy the rest of the application code
COPY . .

# Build the React application
RUN npm run build

# Serve the build files using a lightweight HTTP server
FROM nginx:stable-alpine
COPY --from=builder /app/build /usr/share/nginx/html

# Expose the web server port
EXPOSE 80

# Start the server
CMD ["nginx", "-g", "daemon off;"]
