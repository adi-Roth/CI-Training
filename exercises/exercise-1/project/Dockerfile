# Dockerfile

# Stage 1: Build
FROM python:3.9-slim as builder

# Set working directory
WORKDIR /app

# Copy requirements and install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy application code
COPY app app
COPY tests tests

# Run unit tests
RUN python -m unittest discover -s tests

# Stage 2: Production
FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy application code from the builder stage
COPY --from=builder /app/app app

# Set the entrypoint (for example purposes, a simple Python REPL)
CMD ["python"]
