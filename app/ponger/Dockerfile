# Define the base image
FROM golang:latest AS builder

# Set working directory for build context
WORKDIR /app

# Copy source code
COPY ponger .

# Install dependencies
RUN go mod download

# Build the application 
RUN CGO_ENABLED=0 GOARCH=amd64 GOOS=linux go build -a --installsuffix cgo --ldflags="-s -w" -o ponger .

# Define the final image
FROM alpine:latest

# Set working directory
WORKDIR /app

# Copy the binary
COPY --from=builder /app/ponger .
COPY --from=builder /app/config.yaml .
COPY certs /app/certs

# Expose the port
EXPOSE 8080

# Command to run the application
CMD ["./ponger"]
