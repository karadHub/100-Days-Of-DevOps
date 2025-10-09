# Day 47: Docker Python App

## 🛠️ Task

A python app needed to be Dockerized, and then it needs to be deployed on App Server 2. We have already copied a requirements.txt file (having the app dependencies) under `/python_app/src/` directory on App Server 2. Further complete this task as per details mentioned below:

1. **Create a Dockerfile under `/python_app` directory:**

   - Use any python image as the base image.
   - Install the dependencies using requirements.txt file.
   - Expose the port 6000.
   - Run the server.py script using CMD.

2. **Build an image named `nautilus/python-app` using this Dockerfile.**

3. **Once image is built, create a container named `pythonapp_nautilus`:**

   - Map port 6000 of the container to the host port 8096.

4. **Once deployed, you can test the app using curl command on App Server 2.**
   - `curl http://localhost:8096/`

---

## ✅ Solution

### 📁 1. Create the Dockerfile under `/python_app`

Inside the `/python_app` directory, create a file named `Dockerfile` with the following content:

```Dockerfile
# Use any official Python base image
FROM python:3.10-slim

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY src/requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Copy the server script
COPY src/server.py .

# Expose port 6000
EXPOSE 6000

# Run the server
CMD ["python", "server.py"]
```

---

### 🛠️ 2. Build the Docker image

Run this command from the directory containing `/python_app`:

```bash
docker build -t nautilus/python-app ./python_app
```

---

### 🚀 3. Create and run the container

```bash
docker run -d \
  --name pythonapp_nautilus \
  -p 8096:6000 \
  nautilus/python-app
```

This maps **host port 8096** to **container port 6000**.

---

### ✅ 4. Test the deployment

On **App Server 2**, run:

```bash
curl http://localhost:8096/
```

If everything's set up correctly, you should see the response from `server.py`.

---

## 🔍 Key Notes

- **Python Image**: `python:3.10-slim` is lightweight and suitable for most applications
- **Working Directory**: Set to `/app` for organization
- **Port Mapping**: Container port 6000 → Host port 8096
- **Dependencies**: Installed from `requirements.txt` in the `src/` directory
