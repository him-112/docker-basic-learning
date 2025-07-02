# 🚀 Quick Start Guide - Get Docker Running in 10 Minutes

This guide will get you up and running with Docker quickly so you can start learning immediately.

---

## ⚡ Step 1: Install Docker (5 minutes)

### Windows & Mac
1. Download Docker Desktop from https://docker.com/get-started
2. Run the installer and follow the setup wizard
3. Start Docker Desktop

### Linux (Ubuntu/Debian)
```bash
# Update package index
sudo apt update

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# Add your user to docker group
sudo usermod -aG docker $USER

# Restart your terminal or run:
newgrp docker
```

---

## ✅ Step 2: Verify Installation (1 minute)

Run these commands to confirm Docker is working:

```bash
# Check Docker version
docker --version

# Run the hello-world container
docker run hello-world

# If you see "Hello from Docker!" message, you're ready! 🎉
```

---

## 🎯 Step 3: Your First Real Container (3 minutes)

Let's run a web server and see it in action:

```bash
# Run nginx web server
docker run -d -p 8080:80 --name my-first-web nginx

# Check if it's running
docker ps

# Visit http://localhost:8080 in your browser
# You should see the nginx welcome page!

# Stop and clean up
docker stop my-first-web
docker rm my-first-web
```

**What just happened?**
- `-d`: Run in background (detached mode)
- `-p 8080:80`: Map port 8080 on your computer to port 80 in the container
- `--name my-first-web`: Give the container a friendly name
- `nginx`: The image to run

---

## 🛠️ Step 4: Quick Test Drive (1 minute)

Try these essential commands:

```bash
# See all containers (running and stopped)
docker ps -a

# See all images on your system
docker images

# Get help for any command
docker --help
docker run --help
```

---

## 🎉 You're Ready!

Congratulations! Docker is now installed and working. You can now:

1. **Start Learning**: Go to the [main README](README.md) to begin the full course
2. **Try Beginner Exercises**: Jump to [01-beginner](01-beginner/README.md) for hands-on practice
3. **Quick Reference**: Use the [cheat sheet](cheat-sheets/docker-commands.md) for command references

---

## 🆘 Troubleshooting

### Common Issues and Solutions

#### "Docker daemon not running"
**Windows/Mac**: Make sure Docker Desktop is started
**Linux**: Run `sudo systemctl start docker`

#### "Permission denied" on Linux
Run: `sudo usermod -aG docker $USER` then restart terminal

#### Port already in use
Change the port: `docker run -p 8081:80 nginx` (use 8081 instead of 8080)

#### Can't access localhost:8080
- Make sure the container is running: `docker ps`
- Try 127.0.0.1:8080 instead of localhost:8080
- Check firewall settings

---

## 📚 What's Next?

### Beginner Path (Start here!)
1. [Basic Docker concepts](01-beginner/README.md)
2. [Practice exercises](exercises/README.md#beginner-exercises)
3. [Build your first image](01-beginner/simple-website/)

### Intermediate Path
1. [Docker Compose](02-intermediate/README.md)
2. [Multi-container applications](02-intermediate/web-app-example/)
3. [Advanced Dockerfile techniques](02-intermediate/README.md#advanced-dockerfile-techniques)

### Advanced Path
1. [Production deployment](03-advanced/README.md)
2. [Docker Swarm](03-advanced/README.md#docker-swarm)
3. [Security and monitoring](03-advanced/README.md#security-best-practices)

---

## 💡 Learning Tips

1. **Hands-on Learning**: Docker is best learned by doing. Run the examples!
2. **Start Simple**: Master basic commands before moving to complex orchestration
3. **Read Error Messages**: Docker provides helpful error messages
4. **Use Official Images**: Start with official images from Docker Hub
5. **Clean Up**: Use `docker system prune` regularly to free up space

---

## 🎯 5-Minute Challenge

Before diving into the full course, try this quick challenge:

1. Run a Python container interactively: `docker run -it python:3.9`
2. Inside the container, run: `print("Hello from inside Docker!")`
3. Exit the container and run a Node.js container: `docker run -it node:18`
4. Inside Node, run: `console.log("Docker is awesome!")`

If you completed this challenge, you're ready for the full Docker learning journey!

---

## 🔗 Helpful Links While Learning

- **Docker Hub**: https://hub.docker.com (find images)
- **Docker Docs**: https://docs.docker.com (official documentation)
- **Play with Docker**: https://labs.play-with-docker.com (practice online)
- **This Course**: Start with [README.md](README.md) for the complete learning path

---

**Ready to become a Docker expert? Let's go! 🐳**

[👉 Start the Full Course](README.md) 