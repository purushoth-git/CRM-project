SerpHawk CRM – AWS Deployment

1. Project Overview

SerpHawk CRM is a web-based CRM application with:

Frontend: Next.js

Backend: Python FastAPI

Database: PostgreSQL

Containerization: Docker

Cloud: AWS

The application is deployed with Next.js and FastAPI as separate Docker containers on Amazon EC2, while PostgreSQL is hosted on Amazon RDS.

2. AWS Architecture

                         Internet
                            |
                            v
                 +----------------------+
                 |       AWS VPC        |
                 |                      |
                 |   Public Subnet      |
                 |                      |
                 |  +----------------+  |
                 |  | EC2 Ubuntu     |  |
                 |  | Docker         |  |
                 |  |                |  |
                 |  | Next.js :3000  |  |
                 |  | FastAPI :8000  |  |
                 |  +-------+--------+  |
                 +----------|-----------+
                            |
                       PostgreSQL :5432
                            |
                            v
                 +----------------------+
                 | Amazon RDS PostgreSQL |
                 | Private / No Public   |
                 | Access                |
                 | Database: serphawk    |
                 +----------------------+

3. AWS Services Used

Amazon VPC

Provides the isolated network environment for the application, including public and private subnets and network routing.

Public Subnet

Hosts the EC2 application server so the web application can be reached from the Internet.

Private Subnets

Used for the database layer. RDS is configured without public access.

Amazon EC2

Runs Ubuntu 24.04 and hosts the Next.js and FastAPI Docker containers.

Why EC2? It provides direct control over the application server and is suitable for this small deployment and assignment.

Amazon RDS for PostgreSQL

Hosts the PostgreSQL database separately from the application server.

Why RDS?

Managed PostgreSQL service

Separates database from application compute

AWS manages the underlying database infrastructure

Security Groups

Control network access:

SSH to EC2

Frontend TCP 3000

Backend TCP 8000

PostgreSQL TCP 5432 from the EC2 security group to RDS

RDS should never allow PostgreSQL from 0.0.0.0/0.

Docker

Packages the frontend and backend into separate containers:

serphawk-frontend

serphawk-backend

4. Application Flow

User Browser
     |
     v
Next.js Frontend :3000
     |
     | HTTP API
     v
FastAPI Backend :8000
     |
     | PostgreSQL :5432
     v
Amazon RDS PostgreSQL

The browser communicates with FastAPI. FastAPI handles application logic and database operations.

5. Backend Docker Deployment

Build:

docker build -f Dockerfile.backend -t serphawk-backend .

Run:

docker run -d   --name serphawk-backend   --env-file .env.aws   -p 8000:8000   serphawk-backend

6. Frontend Docker Deployment

Build:

cd frontend

docker build   --build-arg NEXT_PUBLIC_API_BASE_URL=http://YOUR_EC2_PUBLIC_IP:8000   -t serphawk-frontend .

Run:

docker run -d   --name serphawk-frontend   -p 3000:3000   serphawk-frontend

NEXT_PUBLIC_API_BASE_URL tells the Next.js frontend where the FastAPI backend is hosted.

7. Database Configuration

The backend uses an RDS connection string similar to:

postgresql://postgres:<RDS_PASSWORD>@<RDS_ENDPOINT>:5432/serphawk

Store it in an environment file:

DATABASE_URL=postgresql://postgres:<RDS_PASSWORD>@<RDS_ENDPOINT>:5432/serphawk

Never commit .env, .env.aws, passwords, API keys, or other secrets to GitHub.

8. Database Initialization

Create tables:

docker run --rm   --env-file .env.aws   serphawk-backend   python create_tables.py

Seed initial data:

docker run --rm   --env-file .env.aws   serphawk-backend   python seed_db.py

The deployed database was initialized with 29 CRM tables.

9. Verification

Backend health check:

curl http://localhost:8000

Expected:

{"status":"ok","app":"SerpHawk CRM API","docs":"/docs"}

Frontend:

http://<EC2_PUBLIC_IP>:3000

FastAPI documentation:

http://<EC2_PUBLIC_IP>:8000/docs

10. Security

RDS is configured without public access.

PostgreSQL 5432 should only accept traffic from the EC2/application security group.

Do not expose database port 5432 to the Internet.

Keep credentials and API keys out of GitHub.

Restrict SSH access to trusted IP addresses where possible.

The current assignment deployment exposes ports 3000 and 8000 for direct application testing.

11. Deployment Checklist

AWS VPC created

Public subnet created

Private subnets created

EC2 Ubuntu 24.04 created

RDS PostgreSQL created

serphawk database created

Database tables initialized

Docker installed

Backend image built

Frontend image built

Backend container deployed

Frontend container deployed

Application verified in browser

12. Future Improvements

For a production-oriented version:

Add HTTPS and a domain name.

Put an Application Load Balancer in front of the application.

Use private application subnets.

Use AWS Secrets Manager or SSM Parameter Store.

Add CloudWatch monitoring and centralized logs.

Add automated CI/CD with GitHub Actions or Jenkins.

Use multiple application instances for high availability.

Consider ECS or EKS for container orchestration.

13. Repository Structure

CRM-project/
├── Dockerfile.backend
├── frontend/
│   ├── Dockerfile
│   └── ...
├── requirements.txt
├── main.py
├── create_tables.py
├── seed_db.py
└── README.md
