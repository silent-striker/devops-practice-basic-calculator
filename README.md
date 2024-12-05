# DevOps Foundations
### Submission by: Priyatam Reddy Somagattu

## Project Overview
The project is a web based calculator that allows the user to perform arithmetic operations such as addition, subtraction, multiplication and division, along with some advanced calculation like log etc.

### Project Structure
The project consists of two main components a frontend and backend. The frontend is a ReactJS application and the backend is a Flask server that has mathematical operations as endpoints. 

The Main logic of the frontend source code is in `frontend/src/App.js` whereas the backend code resides in `backend/app.py`.

## Docker Implementation
#### **Backend Dockerfile** (Python API):
- **Base Image**: The backend uses the `python:3.13` image, which is the official Python 3.13 image from Docker Hub. It provides a minimal environment to run Python applications.
- **Working Directory**: The working directory is set to `/app/`, where all project files will reside.
- **Dependencies**: The `requirements.txt` file is copied into the container, and then Python dependencies are installed using `pip3 install -r requirements.txt`.
- **Application Code**: The `app.py` file, which contains the Flask application, is copied into the container.
- **Environment Variable**: The `PORT` environment variable is set to `10000`, which is the default port the Flask app will run on.
- **Expose Port**: The container exposes port `10000`, which is used by the Flask API to communicate.
- **Command**: The container will start the Flask server by running `python3 app.py`.

#### **Frontend Dockerfile** (React App):
- **Base Image**: The frontend uses the `node:bullseye-slim` image, which is a lightweight Node.js image based on Debian Bullseye, suitable for running Node.js applications.
- **Working Directory**: The working directory is set to `/app/`.
- **Dependencies**: The `package.json` is copied into the container, followed by installing the required Node.js dependencies using `npm install`.
- **Build React App**: The `npm run build` command is run to create the production-ready build of the React application, which is optimized for deployment.
- **Environment Variables**: 
  - `REACT_APP_BACKEND_URL` is set to `http://localhost`, which points to the backend API.
  - `REACT_APP_BACKEND_PORT` is set to `10000`, which is the backend server's port.
- **Expose Port**: The container exposes port `3000`, which is the default port for the React app.
- **Command**: The container starts a static server using the `serve` tool to serve the built React app on port `3000`.

#### **Explanation for Dockerfile Choices**:
- The **Backend Dockerfile** is optimized for serving a Python Flask app, where dependencies are managed via `requirements.txt` and the app runs on a fixed port.
- The **Frontend Dockerfile** builds and serves a React app using the `serve` tool, which is well-suited for static file serving in a production environment.
- Both Dockerfiles are designed to be lightweight and modular, focusing on the key requirements for running the respective applications.

## Docker Compose YAML Configuration

### **Services:**

The `docker-compose.yml` file defines the following two services:

1. **calculator-frontend**:
   - **Purpose**: This service hosts the React-based frontend of the Calculator application. It is responsible for rendering the user interface in the browser and communicating with the backend API to fetch or process data.
   - **Configuration**:
     - **Image**: The Docker image used is `priyatamsomagattu/calculator-frontend:latest`, which contains the prebuilt production-ready frontend application.
     - **Container Name**: The container is explicitly named `calculator-frontend` for ease of identification during development and debugging.
     - **Ports**: The application is exposed on port `8080` of the host machine and mapped to port `3000` inside the container. This allows users to access the frontend via `http://localhost:8080` or the corresponding hostname.
     - **Dependency**: The service declares a dependency on the `calculator-backend` service using the `depends_on` directive, ensuring that the backend is started before the frontend.
     - **Network**: This service is connected to the `docker-network` network, allowing it to communicate seamlessly with other services on the same network.

2. **calculator-backend**:
   - **Purpose**: This service hosts the Python Flask-based backend API of the Calculator application. It handles the business logic, processes requests from the frontend, and provides appropriate responses.
   - **Configuration**:
     - **Image**: The Docker image used is `priyatamsomagattu/calculator-backend:latest`, which contains the Flask API setup with all required dependencies.
     - **Container Name**: The container is explicitly named `calculator-backend` for consistency and easier troubleshooting.
     - **Ports**: The service exposes port `10000` both inside the container and on the host machine, making the backend API accessible at `http://localhost:10000`.
     - **Environment Variables**: 
       - `PORT`: Set to `10000`, which defines the port on which the Flask application listens.
     - **Network**: Like the frontend, the backend is also connected to the `docker-network` network for inter-service communication.

---

### **Networking:**

- **Custom Network**: 
  - The `docker-network` is a user-defined bridge network created to isolate and manage communication between the services (`calculator-frontend` and `calculator-backend`). 
  - The bridge network allows containers to resolve each other's names and communicate via Docker's built-in DNS. For example:
    - The frontend can access the backend by referring to it as `calculator-backend` (the container name) within the network.
  - This ensures that communication is seamless without requiring external hostnames or IP addresses.
  - By isolating services within a custom network, external containers or processes cannot interfere with the application unless explicitly connected.

---

### **Volumes:**

- No volume mounts are defined in this configuration. 
  - **Impact**: This means the containers are ephemeral, and all data or application state is stored inside the container. Upon stopping or removing the containers, any changes made inside the container (outside the image files) will be lost.
  - If persistent data storage were required (e.g., for logs or a database), volumes could be defined to map specific directories from the container to the host machine.

---

### **Environment Variables:**

- The `calculator-backend` service defines one environment variable:
  - **PORT**:
    - This environment variable is set to `10000`, specifying the port on which the backend Flask application will listen. 
    - It ensures consistency across environments and allows easier configuration changes if the port needs to be updated in the future.

---

### **Service Interaction and Configuration:**

- **Service Dependencies**: 
  - The `depends_on` directive ensures that the `calculator-backend` service starts before the `calculator-frontend`. However, this only ensures the startup sequence and does not guarantee that the backend will be ready to accept requests. Additional health checks can be added to ensure the backend's readiness.

- **Inter-Service Communication**:
  - The frontend (`calculator-frontend`) interacts with the backend (`calculator-backend`) through the `docker-network`. Within the network, the frontend can use the backend's container name (`calculator-backend`) to make HTTP requests to the API (e.g., `http://calculator-backend:10000`).

- **Ports Exposure**:
  - Exposing ports for both services allows them to be accessible outside the Docker network:
    - The frontend is accessible via `http://localhost:8080` on the host.
    - The backend API is accessible via `http://localhost:10000`.

- **Scalability and Modularity**:
  - Each service is encapsulated in its own container, following a microservices approach. This modularity makes it easier to update, scale, and debug individual services without affecting others.

---

This `docker-compose.yml` file is designed for a simple two-service architecture, providing clear separation of concerns between the frontend and backend, with efficient communication and container management via Docker Compose. Future enhancements might include volume definitions for persistent data, health checks for better orchestration, and environment variable files for secure configuration management.

## CI/CD Pipeline (YAML Configuration)

### **Triggers:**

- The pipeline is triggered on the following events:
  - **Push to the `main` branch**: Automatically initiates the pipeline when changes are pushed to the main branch.
  - **Pull Request to the `main` branch**: Runs the pipeline to validate changes before merging them into the main branch.

---

### **Pipeline Stages:**

The CI/CD pipeline consists of three main stages:

1. **Frontend Stage**:
   - **Purpose**: Build the frontend React application.
   - **Steps**:
     1. **Checkout Code**:
        - Uses the `actions/checkout@v3` action to clone the repository.
     2. **Set Up Node.js**:
        - Configures the environment using Node.js version 16.
     3. **Install Dependencies**:
        - Runs `npm install` in the `frontend/` directory to install required packages.
     4. **Build React App**:
        - Builds the production-ready frontend using `npm run build`.

2. **Backend Stage**:
   - **Purpose**: Set up the backend Python Flask application.
   - **Steps**:
     1. **Checkout Code**:
        - Clones the repository.
     2. **Set Up Python**:
        - Configures the environment using Python version 3.9.
     3. **Install Dependencies**:
        - Installs the backend dependencies by running `pip3 install -r backend/requirements.txt`.

3. **Docker Stage**:
   - **Purpose**: Build Docker images for both the frontend and backend, then push them to Docker Hub.
   - **Dependencies**:
     - This stage depends on the successful completion of the `frontend` and `backend` stages.
   - **Steps**:
     1. **Checkout Code**:
        - Ensures the repository code is available for the Docker build process.
     2. **Log in to Docker Hub**:
        - Uses `docker/login-action@v2` to authenticate to Docker Hub using credentials stored in GitHub secrets.
     3. **Build Docker Images**:
        - Builds Docker images for the frontend (`calculator-frontend`) and backend (`calculator-backend`) using the respective Dockerfiles.
     4. **Push Docker Images**:
        - Pushes the built images to Docker Hub under the repository specified by the secret `DOCKER_USERNAME`.

---

### **Docker Image Build and Push Process:**

- **Frontend Docker Image**:
  - Built using the Dockerfile in the `frontend/` directory.
  - Tagged as `latest` and pushed to Docker Hub under the repository `${{ secrets.DOCKER_USERNAME }}/calculator-frontend`.

- **Backend Docker Image**:
  - Built using the Dockerfile in the `backend/` directory.
  - Tagged as `latest` and pushed to Docker Hub under the repository `${{ secrets.DOCKER_USERNAME }}/calculator-backend`.

---

### **Automation and Deployment Process:**

- **Continuous Integration**:
  - The pipeline ensures code validity by automatically building both the frontend and backend upon any updates to the `main` branch.
  - The modular design of the stages ensures errors are isolated to specific services, simplifying troubleshooting.

- **Continuous Deployment**:
  - Upon successful build and push of Docker images, these images can be used to deploy the updated application via Docker Compose or any other container orchestration tool.

---

This CI/CD pipeline ensures an automated, repeatable, and efficient process for building, testing, and deploying the Calculator application, reducing manual overhead and enhancing software delivery speed and reliability.

## Assumptions

1. **Shared Network for Containers**:
   - Both the **frontend** and **backend** services are assumed to run on the same Docker network (`docker-network`) to simplify communication between them. 
   - This eliminates the need for external DNS or complex networking configurations.

2. **Environment Variables for Frontend**:
   - Environment variables for the **frontend** service, such as `REACT_APP_BACKEND_URL` and `REACT_APP_BACKEND_PORT`, are set during the **build stage**.
   - React's build process injects these values into static files (e.g., HTML, JS), making them immutable at runtime. This decision was made with the understanding that runtime environment changes would not affect these static files.

3. **Container Hosting on the Same Machine**:
   - Both containers are expected to run on the same physical or virtual machine. This avoids cross-host communication challenges and simplifies the deployment environment.

4. **Port Binding**:
   - The **frontend** service maps port `3000` inside the container to port `8080` on the host, assuming the host machine has this port available for binding.
   - The **backend** service binds its port `10000` directly, expecting no conflicts with other services.

5. **No Persistent Data Storage**:
   - No volumes were defined in `docker-compose.yml`, implying that any state or logs generated within the containers are ephemeral. This is acceptable since the focus is on building and deploying stateless services.

6. **Pre-Configured Docker Hub Secrets**:
   - The CI/CD pipeline assumes that secrets for Docker Hub (`DOCKER_USERNAME` and `DOCKER_PASSWORD`) are correctly configured in GitHub Actions.

7. **Python and Node.js Versions**:
   - The pipeline assumes compatibility with Python `3.9` for the backend and Node.js `16` for the frontend.

8. **Build Dependencies Installed**:
   - The Dockerfiles assume that all necessary dependencies (e.g., `pip3`, `npm`) are installed and functional in the base images (`python:3.13` for the backend and `node:bullseye-slim` for the frontend).

These assumptions help establish a controlled environment for building, deploying, and running the application with minimal configuration overhead.

## Lessons Learned

### Challenges Faced
1. **Docker Networks and Inter-Container Communication**:
   - A significant challenge was ensuring seamless communication between containers. Configuring a shared network (`docker-network`) and debugging network-related issues required an understanding of Docker's networking model.
   
2. **API Calls from Browser to Backend**:
   - API requests made by the **frontend** (running in the browser) to the **backend** service were initially unreachable. This issue stemmed from the fact that the browser operates outside the Docker network, requiring explicit configuration of host machine ports to enable access.

### Key Learnings
1. **Consistent Dependency Management**:
   - Containerization with Docker ensures consistent packaging of dependencies, enabling the application to run reliably across different machines and environments. This minimizes "it works on my machine" scenarios.

2. **Streamlined Code Integration with CI/CD**:
   - Implementing pipelines significantly improved the code integration process:
     - Ensured quality by automating tests and builds, catching potential issues early.
     - Reduced human error by automating repetitive tasks such as building and pushing Docker images.

3. **Automation Benefits**:
   - Automation in CI/CD pipelines reduced manual intervention, improved deployment speed, and maintained consistency in the build process. This highlighted the value of investing time in setting up robust pipelines.

4. **Docker Networking Concepts**:
   - Gained deeper insights into Docker's networking model, such as the use of `bridge` networks, port mappings, and the difference between internal and external network traffic.

5. **Runtime vs Build-Time Configuration**:
   - Learned that tools like React inject environment variables into static files at build time, making them immutable during runtime. This emphasized the importance of correctly setting environment variables during the build stage for frontend services.

Overall, this experience reinforced the importance of containerization and automation in modern software development, making deployment more reliable and scalable.

## Future Improvements

### Enhancements to Dockerfiles
1. **Use of Environment Variables and Build Arguments**:
   - Introduce `ARG` and `ENV` directives to make the Dockerfiles more customizable, enabling dynamic configuration at both build and runtime. For example:
     - Use `ARG` for build-specific variables like backend API URLs.
     - Use `ENV` for runtime variables, making the containers more versatile across different environments (e.g., development, testing, production).

2. **Optimize Layering**:
   - Reorder instructions to take advantage of Docker’s layer caching for faster builds.
   - Separate installation of dependencies from application source code to avoid unnecessary rebuilding when only source code changes.

3. **Multi-Stage Builds**:
   - Implement multi-stage builds for the frontend and backend to reduce image size by separating the build environment from the runtime environment.

### Enhancements to `docker-compose.yml`
1. **Volume Mounts for Development**:
   - Use volume mounts to sync local code changes to containers during development for rapid iteration.
   - Example: Mount source code directories for live reload of the frontend or backend.

2. **Environment File Support**:
   - Use an `.env` file to externalize environment variables, improving security and maintainability.

3. **Health Checks**:
   - Add `healthcheck` configurations to monitor the health of containers and ensure dependent services wait until others are ready.

### Enhancements to CI/CD Pipeline
1. **Add Test Stages**:
   - Introduce unit testing for both frontend and backend code to catch errors early in the pipeline.
     - Use Jest for frontend React components.
     - Use pytest for backend Python APIs.

2. **Code Linting and Formatting**:
   - Include stages for code linting (e.g., ESLint for frontend, Flake8 for backend) to enforce coding standards.

3. **Security Scanning**:
   - Add a security scanning stage using tools like Docker’s built-in `docker scan` or third-party tools like Snyk to identify vulnerabilities in dependencies and Docker images.

4. **Integration Testing**:
   - Implement integration tests to validate interactions between the frontend and backend.
   - Use a testing framework like Cypress for end-to-end testing.

5. **Production Deployment**:
   - Expand the pipeline to include deployment stages to production or staging environments using tools like Kubernetes, AWS ECS, or Azure Container Instances.

### Additional Functionalities for Calculator Application
1. **Improved Features**:
   - Add advanced calculator functionalities like scientific calculations, history tracking, or user authentication.

2. **Dynamic Backend Configuration**:
   - Enhance the backend to support multiple endpoints or versions, improving scalability and flexibility.

3. **Pipeline Stages**:
   - Introduce stages to deploy separate environments (e.g., staging, testing, production).
   - Add rollback strategies for deployments to ensure stability in production.

By implementing these improvements, the workflow and application can become more robust, scalable, and adaptable to evolving requirements.