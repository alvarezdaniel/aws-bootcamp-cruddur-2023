# Use a smaller base image
FROM python:3.10-slim as base

# Set working directory
WORKDIR /backend-flask

# Install build dependencies
RUN apt-get update && apt-get install -y build-essential

# Install production dependencies
FROM base as dependencies
COPY --chown=root:root requirements.txt .
RUN pip3 install --no-cache-dir -r requirements.txt

# Copy source code
FROM dependencies as source
COPY . .

# Set environment variables
ENV FLASK_ENV=development
ENV PORT=4567

# Expose port
EXPOSE ${PORT}

# Run the app
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]