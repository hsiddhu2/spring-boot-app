# Spring Boot Application Deployment to Amazon EC2 using GitHub Actions

This project demonstrates how to deploy a Spring Boot application to an Amazon EC2 instance using GitHub Actions for continuous integration and deployment (CI/CD).

## Project Setup

### Prerequisites

- Java 17
- Maven
- Spring Boot
- Amazon EC2 instance
- GitHub repository

### Application Setup

1. **Spring Boot Application**: The application is a simple Spring Boot project with a home page.

2. **Project Structure**:
    - `src/main/java/com/cloudwithharry/springapp/SpringAppApplication.java`: Main application class.
    - `src/main/java/com/cloudwithharry/springapp/HomePageController.java`: Controller for the home page.
    - `src/main/resources/templates/index.html`: Thymeleaf template for the home page.
    - `pom.xml`: Maven configuration file.

### Server Setup

1. **Create Deployment Directory**:
    - Create a directory on the EC2 instance to store the application files.

    ```sh
    mkdir -p /home/ec2-user/spring-app
    ```

2. **Systemd Service File**:
    - Create a systemd service file to manage the Spring Boot application.

    ```sh
    sudo nano /etc/systemd/system/spring-app.service
    ```

    - Add the following content to the service file:

    ```ini
    [Unit]
    Description=Spring Boot Application

    [Service]
    User=ec2-user
    ExecStart=/usr/bin/java -jar /opt/spring-app/spring-app.jar
    SuccessExitStatus=143

    [Install]
    WantedBy=multi-user.target
    ```

3. **Start and Enable the Service**:

    ```sh
    sudo systemctl daemon-reload
    sudo systemctl start spring-app
    sudo systemctl enable spring-app
    ```

### CI/CD Setup

1. **GitHub Secrets**:
    - Store the EC2 key pair and public DNS in GitHub secrets:
        - `EC2_KEY_PAIR`: The private key of the EC2 instance.
        - `EC2_PUBLIC_DNS`: The public DNS of the EC2 instance.

2. **GitHub Actions Workflow**:
    - Create a `.github/workflows/cicd.yml` file with the following content:

    ```yaml
    name: CI/CD Pipeline

    on:
      push:
        branches:
          - main

    jobs:
      build:
        runs-on: ubuntu-latest

        steps:
        - name: Checkout code
          uses: actions/checkout@v2

        - name: Set up JDK 17
          uses: actions/setup-java@v2
          with:
            java-version: '17'

        - name: Build with Maven
          run: mvn clean package

        - name: Deploy to EC2
          uses: appleboy/ssh-action@v0.1.5
          with:
            host: ${{ secrets.EC2_PUBLIC_DNS }}
            username: ec2-user
            key: ${{ secrets.EC2_KEY_PAIR }}
            script: |
              sudo systemctl stop spring-app
              rm -rf /opt/spring-app/*
              scp -i ${{ secrets.EC2_KEY_PAIR }} target/spring-app-0.0.1-SNAPSHOT.jar ec2-user@${{ secrets.EC2_PUBLIC_DNS }}:/opt/spring-app/spring-app.jar
              sudo systemctl start spring-app
    ```

## Running the Application

1. **Build the Application**:

    ```sh
    mvn clean package
    ```

2. **Deploy the Application**:
    - Push the changes to the `main` branch to trigger the GitHub Actions workflow.

3. **Access the Application**:
    - Open a web browser and navigate to `http://<EC2_PUBLIC_DNS>` to see the application running.

## Conclusion

This project demonstrates how to set up a Spring Boot application, configure an EC2 instance, and use GitHub Actions for CI/CD to deploy the application to the EC2 instance.
