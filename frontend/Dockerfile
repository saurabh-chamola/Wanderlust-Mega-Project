# ------------------- Stage 1: Build Stage ------------------------------
FROM node:21 AS frontend-builder

# Set the working directory
WORKDIR /app

# Copy dependency files
COPY package*.json ./

# Install all dependencies
RUN npm install

# Copy the rest of your application code
COPY . .

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

# Expose the port your app runs on (Vite default build is static, so any port works)
EXPOSE 5173

# Run the production build using serve
CMD ["serve", "-s", ".", "-l", "5173"]
