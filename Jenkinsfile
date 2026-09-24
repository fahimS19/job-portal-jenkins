pipeline {
    agent any

    environment {
        PROJECT_NAME = "react-job-portal"
        FRONTEND_PORT = "5173"
        BACKEND_PORT = "4000"
    }

    stages {

        stage('Clone Repository') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/fahimS19/job-portal-jenkins-final',
                    credentialsId: 'react-portal-private-repo'

                sh 'ls -lah'
            }
        }

        stage('Create Dockerfiles') {
            steps {

                dir('frontend') {
                    writeFile file: 'Dockerfile', text: """
FROM node:24-slim AS builder

WORKDIR /bjit

COPY package*.json ./

RUN npm install

FROM node:24-alpine AS production

WORKDIR /bjit

COPY --from=builder /bjit/node_modules ./node_modules
COPY --from=builder /bjit/package*.json ./

COPY . .

EXPOSE 5173

CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
"""
                }

                dir('backend') {
                    writeFile file: 'Dockerfile', text: """
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./

RUN npm ci --only=production

FROM node:20-alpine AS runner

WORKDIR /app

COPY --from=builder /app/node_modules ./node_modules

COPY . .

EXPOSE 4000

CMD ["node", "server.js"]
"""
                }
            }
        }

        stage('Create Compose File') {
            steps {

                writeFile file: 'docker-compose.yml', text: """
services:

  frontend:
    build:
      context: ./frontend
    image: react-job-portal-frontend:latest
    container_name: react-job-portal-fe-compose
    ports:
      - "8081:5173"
    depends_on:
      - backend

  backend:
    build:
      context: ./backend
    image: react-job-portal-backend:latest
    container_name: react-job-portal-backend-compose
    ports:
      - "8082:4000"
    environment:
      PORT: 4000
      DB_URL: mongodb://admin:securepassword@mongo:27017/jobportal?authSource=admin
      JWT_SECRET_KEY: fgjfgsudgfudnhfousidfnewuikfmlewf
      JWT_EXPIRE: 7d
      COOKIE_EXPIRE: 7
      NODE_ENV: development
      CLOUDINARY_API_KEY: 819224586478434
      CLOUDINARY_API_SECRET: yr_TbbT_JDG2Xfu4KcrrcykeJvA
      CLOUDINARY_CLOUD_NAME: dtgsdprcl
      FRONTEND_URL: http://localhost:8081
    depends_on:
      - mongo

  mongo:
    image: mongo:6
    container_name: common-mongodb-compose
    ports:
      - "27017:27017"
    environment:
      MONGO_INITDB_ROOT_USERNAME: admin
      MONGO_INITDB_ROOT_PASSWORD: securepassword

"""
            }
        }

        stage('Deploy with Compose') {
            steps {
                sh '''
                    docker compose down
                    docker compose up -d --build
                '''
            }
        }

        stage('Check Containers') {
            steps {
                sh '''
                    docker compose ps
                    docker compose logs --tail=100
                '''
            }
        }
    }
}

