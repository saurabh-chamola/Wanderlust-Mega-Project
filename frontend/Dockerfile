# ------------------- Stage 1: Build Stage ------------------------------
FROM node:21 AS frontend-builder

# Set the working directory
WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install all dependencies
RUN npm install

# Copy all source files, including .env.docker
COPY . .

# Rename .env.docker to .env so Vite picks it up during build
RUN cp .env.docker .env

# Build the production-ready static files using Vite
RUN npm run build

# ------------------- Stage 2: Production Stage -------------------------
FROM node:21-slim

# Set the working directory
WORKDIR /app

# Install a lightweight static server (like serve)
RUN npm install -g serve

# Copy only the built production assets from the previous stage
COPY --from=frontend-builder /app/dist ./ 

# Expose the port your app will be served on
EXPOSE 5173

# Serve the built frontend
CMD ["serve", "-s", ".", "-l", "5173"]
