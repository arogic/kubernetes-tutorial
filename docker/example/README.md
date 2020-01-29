# Build Docker Image
docker build -t nginx-test .

# Run Docker Image on port 8081
docker run -p 8081:80 -d nginx-test

# Test access to Website
curl http://localhost:8081
