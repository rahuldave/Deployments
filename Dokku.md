# Dokku



Let's go through a tutorial on using Dokku as a Platform as a Service (PaaS). Dokku is an open-source PaaS that allows you to deploy and manage applications easily on your own server. It's similar to Heroku but self-hosted.

### Prerequisites

- A server (e.g., a virtual private server from DigitalOcean, AWS, etc.)
- A domain name (optional but recommended)
- Basic knowledge of using the terminal and Git

### Step 1: Set Up Your Server

First, you'll need a server running Ubuntu. For this tutorial, we'll use Ubuntu 20.04.

1. **Connect to your server:**

   ```bash
   ssh user@your-server-ip
   ```

2. **Update the package list and install necessary packages:**

   ```bash
   sudo apt-get update
   sudo apt-get install -y software-properties-common
   ```

### Step 2: Install Dokku

1. **Add the Dokku repository:**

   ```bash
   wget https://dokku.com/install/v0.26.10/bootstrap.sh
   sudo DOKKU_TAG=v0.26.10 bash bootstrap.sh
   ```

2. **Follow the on-screen instructions:**

   - During the installation, you'll be asked to enter your domain name or the server IP address.
   - Set up an SSH key for your Dokku user. You can either provide a public SSH key or let Dokku generate one for you.

### Step 3: Set Up a Domain (Optional)

If you have a domain, you can set up DNS to point to your server. Create an A record pointing your domain (e.g., `yourdomain.com`) to your server's IP address.

### Step 4: Deploying Your First Application

Let's deploy a simple Flask application.

1. **Create a sample Flask application:**

   **app.py:**

   ```bash
   from flask import Flask
   app = Flask(__name__)
   
   @app.route('/')
   def hello_world():
       return 'Hello, World!'
   
   if __name__ == '__main__':
       app.run(host='0.0.0.0')
   ```

   **requirements.txt:**

   ```python
   Flask==2.0.2
   ```

   **Procfile:**

   ```bash
   web: python app.py
   ```

2. **Initialize a Git repository and commit your code:**

   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   ```

3. **Create a new Dokku app:**

   ```bash
   ssh dokku@your-server-ip apps:create myapp
   ```

4. **Add your Dokku server as a remote:**

   ```bash
   git remote add dokku dokku@your-server-ip:myapp
   ```

5. **Deploy your application:**

   ```bash
   git push dokku master
   ```

   After pushing, Dokku will build and deploy your application. You can access it at `http://your-server-ip:port`or `http://yourdomain.com` if you set up a domain.

### Step 5: Managing Your Application

#### Environment Variables

You can set environment variables using Dokku's `config` command:

```bash
ssh dokku@your-server-ip config:set myapp MY_ENV_VAR=value
```

### Using Dockerfiles with Dokku

#### Step 1: Create a Dockerfile

If you haven't already, create a `Dockerfile` in your application's root directory. Here’s an example Dockerfile for a Flask application:

**Dockerfile**

```dockerfile
# Use an official Python runtime as a parent image
FROM python:3.9-slim

# Set the working directory in the container
WORKDIR /app

# Copy the current directory contents into the container at /app
COPY . /app

# Install any needed packages specified in requirements.txt
RUN pip install --no-cache-dir -r requirements.txt

# Make port 5000 available to the world outside this container
EXPOSE 5000

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```

#### Step 2: Commit Your Dockerfile

Add the `Dockerfile` to your Git repository and commit it.

```dockerfile
git add Dockerfile
git commit -m "Add Dockerfile"
```

#### Step 3: Push to Dokku

Push your changes to Dokku as usual:

```dokku
git push dokku master
```

Dokku will detect the presence of the Dockerfile and use it to build your application instead of the default buildpacks. It will output the logs from the Docker build process, and you can see each step being executed as defined in your Dockerfile.

### Customizing Docker Builds

Using a Dockerfile gives you full control over the build process. You can:

- **Install system dependencies:** Add `RUN` commands to install packages using your Linux distribution's package manager.

- **Include custom software:** Copy binaries or other software into the container.

- **Dockerfiles in Dokku:** Allow for custom build processes and environments for your applications.

- **Multi-Stage Builds:** Help create efficient and smaller Docker images.

- **Docker Compose:** Although not natively supported, can be managed manually for multi-service setups.

- **Debugging:** Use Dokku logs and direct container access for troubleshooting.

  Using Dockerfiles with Dokku gives you the flexibility and control to define exactly how your application should be built and run, making it easier to manage complex deployments and ensure consistency across environments.