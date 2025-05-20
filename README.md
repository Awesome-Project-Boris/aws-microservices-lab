# Coffee Suppliers Microservices

This repository contains two Node.js/Express microservices for a Coffee Suppliers application:

* **Customer service** (port 80): exposes read-only endpoints and a web UI for browsing suppliers.
* **Employee (Admin) service** (port 8081): exposes full CRUD endpoints and a web UI under `/admin` for managing suppliers.

Both services use MySQL (via `mysql2`) and EJS for server-side rendering.

---

## Project Structure

```
/
├── customer/
│   ├── app/
│   │   ├── config/config.js
│   │   ├── controller/
│   │   ├── models/
│   │   └── views/
│   ├── index.js
│   └── Dockerfile
└── employee/
    ├── app/
    │   ├── config/config.js
    │   ├── controller/
    │   ├── models/
    │   └── views/
    ├── index.js
    └── Dockerfile
```

---

## Prerequisites

1. **Node.js** (v14+) and `npm`
2. **MySQL** server (v5.7+) running locally or remotely
3. **Docker & Docker CLI**
4. (Optional) **AWS CLI** configured if you plan to push images to ECR and deploy to ECS

---

## Configuration

Each service reads these environment variables:

| Variable          | Description         | Example    |
| ----------------- | ------------------- | ---------- |
| APP\_DB\_HOST     | MySQL host          | localhost  |
| APP\_DB\_USER     | MySQL username      | root       |
| APP\_DB\_PASSWORD | MySQL password      | hunter2    |
| APP\_DB\_NAME     | MySQL database name | coffee\_db |

Set them in your shell or environment before starting or building the services.

```
export APP_DB_HOST=localhost
export APP_DB_USER=root
export APP_DB_PASSWORD=hunter2
export APP_DB_NAME=coffee_db
```

---

## Running Locally

1. **Clone** the repo and `cd` into it:

   ```bash
   git clone https://github.com/<your-org>/coffee-suppliers-microservices.git
   cd coffee-suppliers-microservices
   ```

2. **Install dependencies** for each service:

   ```bash
   cd customer
   npm install
   cd ../employee
   npm install
   ```

3. **Start MySQL**, create the `coffee_db` schema and `suppliers` table as needed.

4. **Run the services** in separate terminals:

   * Customer service (port 80):

     ```bash
     cd customer
     npm start
     ```
   * Employee service (port 8081):

     ```bash
     cd employee
     npm start
     ```

5. **Browse:**

   * [http://localhost/](http://localhost/) → Customer UI
   * [http://localhost:8081/admin](http://localhost:8081/admin) → Employee (Admin) UI

---

## Docker

### Build Images

```bash
# Customer service
cd customer
docker build -t customer:latest .

# Employee service
cd ../employee
docker build -t employee:latest .
```

### Run Containers Locally

```bash
# Customer (port 80)
docker run -d \
  -e APP_DB_HOST \
  -e APP_DB_USER \
  -e APP_DB_PASSWORD \
  -e APP_DB_NAME \
  -p 80:80 \
  --name coffee-customer \
  customer:latest

# Employee (port 8081)
docker run -d \
  -e APP_DB_HOST \
  -e APP_DB_USER \
  -e APP_DB_PASSWORD \
  -e APP_DB_NAME \
  -p 8081:8081 \
  --name coffee-employee \
  employee:latest
```

---

## AWS Deployment (Optional)

1. **ECR:**

   ```bash
   aws ecr create-repository --repository-name customer --region us-east-1
   aws ecr create-repository --repository-name employee --region us-east-1
   ```
2. **Tag & Push:**

   ```bash
   aws ecr get-login-password --region us-east-1 | docker login --username AWS --password-stdin 821374705668.dkr.ecr.us-east-1.amazonaws.com

   docker tag customer:latest 821374705668.dkr.ecr.us-east-1.amazonaws.com/customer:latest
   docker push 821374705668.dkr.ecr.us-east-1.amazonaws.com/customer:latest

   docker tag employee:latest 821374705668.dkr.ecr.us-east-1.amazonaws.com/employee:latest
   docker push 821374705668.dkr.ecr.us-east-1.amazonaws.com/employee:latest
   ```
3. **ECS Fargate:**

   ```bash
   aws ecs create-cluster --cluster-name microservices-serverlesscluster
   # Deploy services/tasks referencing the above images and LabVPC/PublicSubnet1,2.
   ```

---

## References

* [Express](https://expressjs.com/)
* [Docker CLI](https://docs.docker.com/engine/reference/commandline/cli/)
* [AWS CLI ECS](https://docs.aws.amazon.com/cli/latest/reference/ecs/index.html)
