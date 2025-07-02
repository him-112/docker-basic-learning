# Use official nginx image as base image
# Alpine version is smaller and more secure
FROM nginx:alpine

# Set maintainer information (optional but good practice)
LABEL maintainer="your-email@example.com"
LABEL description="My first Docker website"

# Copy our HTML file to nginx default directory
# This replaces the default nginx welcome page
COPY index.html /usr/share/nginx/html/

# Expose port 80 to allow connections
# This is for documentation - nginx image already exposes port 80
EXPOSE 80

# No CMD needed - nginx image already has CMD to start nginx
# The default command will start nginx automatically 