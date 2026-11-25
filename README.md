# Company Landing Page

A professional and responsive company website built using HTML and CSS.

## Technologies Used
- HTML5
- CSS3
- Docker
- Nginx
- Jenkins
- GitHub

## Project Structure

company-website/
- index.html
- style.css
- Dockerfile
- Jenkinsfile
- README.md

## Docker

Docker is used to create a container image of the website.

Nginx is used to serve the website.

## CI/CD Pipeline

1. Developer updates code in GitHub.
2. Jenkins fetches the latest code.
3. Build server creates a Docker image.
4. Image is pushed to Docker Hub.
5. Deployment server pulls the image.
6. Docker runs the updated website.

## Application Port

9090
