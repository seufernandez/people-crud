# Usage

## Running via Tilt

[Tilt](https://tilt.dev/) is a development tool used for orchestrating Kubernetes applications locally.

To run the application, you must have Tilt [installed](https://docs.tilt.dev/install.html), and then simply run the tilt up command in the project's root folder where the Tiltfile is located.

Tilt has a web interface for better application visualization:

![image](https://github.com/user-attachments/assets/0e2da332-c04e-4d4e-a246-a6c047136398)

## Running manually

### Services
The first step is to start Redis, RabbitMQ, and MongoDB by running docker-compose up -d in the project's root folder.

### Backend
```bash
# Enter the backend folder
cd backend

# Create the environment variables file
cp .env.example .env

# Install dependencies
npm install

# Run the backend
npm start
```

### Frontend
```bash
# Enter the frontend folder
cd frontend

# Create the environment variables file
cp .env.example .env

# Install dependencies
npm install

# Run the frontend
npm start
```

# Backend
For the backend, I used NodeJs.

### Collections

The main collection is Users, defined as in the example below:
```json
{
    "_id": "6564d13f6a76b0f57933e45a",
    "firstName": "John",
    "lastName": "Doe",
    "addresses": [
        {
            "street": "Cristo Rei",
            "city": "São Paulo",
            "state": "São Paulo",
            "zipCode": "23242-423",
            "_id": "6564d73e6a76b0f57933e470"
        }
    ],
    "dateOfBirth": "2023-11-02T17:26:11.561Z",
    "email": "email0@gmail.com",
    "documentNumber": "063.548.329-30",
    "phoneNumbers": [
        {
            "number": "(99) 9999-9999",
            "type": "work",
            "_id": "6564d79e850b5928299a28f1"
        },
        {
            "number": "(99) 9999-9999",
            "type": "personal",
            "_id": "6564d79e850b5928299a28f2"
        }
    ],
    "__v": 0
}
```

Additionally, there's the Logs collection:

```json
{
    "_id": "6564d79e850b5928299a28f9",
    "requestTime": "2023-11-27T17:53:34.111Z",
    "responseTime": "2023-11-27T17:53:34.145Z",
    "method": "PUT",
    "url": "/6564d13f6a76b0f57933e45a/phoneNumbers",
    "statusCode": 200,
    "userAgent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/118.0.0.0 Safari/537.36 OPR/104.0.0.0",
    "body": {
        "phoneNumbers": [
            {
                "number": "(99) 9999-9999",
                "type": "work",
                "_id": "6564d79e850b5928299a28f1"
            },
            {
                "number": "(99) 9999-9999",
                "type": "personal",
                "_id": "6564d79e850b5928299a28f2"
            }
        ]
    },
    "params": {
        "id": "6564d13f6a76b0f57933e45a"
    },
    "query": {},
    "__v": 0
}
```

### Authentication
Authentication is done through a keyword present in the API_KEY environment variable. To validate a keyword, a request is made to the route http://localhost:3001/auth/validate, passing the apiKey in the body, and then the API returns a JWT token (which contains the keyword tested by the user). This token is saved in Redis, and for each request on routes protected by the authentication middleware, a query is made to Redis to verify if the token is present and if the keyword is valid. Every request protected by the middleware must contain the token key in the header. Example of body:

```json
{
   "apiKey": "KEY_WORD"
}
```

### Input Validators
To validate inputs received by the API, I used express-validator and created validation functions using regex to verify fields such as email, phone number, document, and ZIP code.

### Middlewares
Two middlewares were implemented, which I explain in more detail below in the extras section:
- Authentication middleware
- Log middleware

### Automated Tests
The application has both unit tests and e2e tests, testing the main system functionalities. In total, there are 21 test cases for validation functions (in the case of unit tests) and requests for the main routes (in the case of e2e)

To run, simply use the npm run test command (in the backend folder)

![tests](https://github.com/user-attachments/assets/71666db2-bc22-4835-93bf-a1e8249505ec)

### Documentation (Swagger)
The complete API documentation can be accessed at http://localhost:3001/api-docs

![swagger](https://github.com/user-attachments/assets/86ff9d24-224a-482a-b1ec-36cc3f38c6e8)
![swagger2](https://github.com/user-attachments/assets/bbfdd562-2b30-4cc1-94a5-2e228a78985c)


### Queue (RabbitMQ)
RabbitMQ is used for writing a new record in the Logs collection. For each request that passes through the logs middleware, a message is published to be consumed asynchronously by the consumer, and then generate the new record in the collection. The idea is that writing the new log doesn't create a bottleneck in standard requests. The consumer can be turned on or off through the LOGGER environment variable, which by default comes with the value ON.

### Redis
Redis was used to implement a cache system to validate user authentication. Validation is done by searching for the token in Redis memory and extracting the keyword from it. Each time a user tries to validate an apiKey, a new record is saved in Redis.

### Docker compose
4 services are present in docker-compose.yml:
- Redis (port 6379)
- RabbitMQ (port 5672)
- RabbitMQ Panel (port 15672)
- MongoDB (port 27017)

# Frontend
For the frontend, I chose to use ant design as a styling library. The layout is based on 4 main sections:
- An input to validate the API keyword
- The users table, with listing, creation, update, and deletion
- Modal for creating a new user
- The logs table for listing records

Additionally, the design was made with responsiveness in mind to adapt to any screen size. Form validators, input masks, filters, and sorting in tables were also added, along with updating and deleting users, phone numbers, and addresses directly in the table itself, facilitating the user experience.

Screenshots for better visualization:
![front1](https://github.com/user-attachments/assets/17f3cfad-2ef0-449c-9b43-3130f45bda6e)

![front2](https://github.com/user-attachments/assets/8026b15d-db42-4133-9ce1-5dec62631ec9)

![front3](https://github.com/user-attachments/assets/34a2c9a5-dcd4-467e-bba3-45b916c4d992)

![front4](https://github.com/user-attachments/assets/8305a359-851f-49cf-8371-73b6ad416108)

![front5](https://github.com/user-attachments/assets/b9b832be-6677-4ba3-aa01-302cfae80f1b)

![front6](https://github.com/user-attachments/assets/3394f622-c650-4146-9ed4-100b734d698e)

![front7](https://github.com/user-attachments/assets/74d428d3-296e-4bf5-ad7a-83c17189b9d7)

![front8](https://github.com/user-attachments/assets/b6c5c6c8-0a52-4811-b66a-dc27749a49ef)

## Tasks

### Backend
- [x] Create initial setup
- [x] Create database connection
- [x] Create database models
- [x] Create routes
    - [x] Create
    - [x] Update
    - [x] Read One
    - [x] Read Many
    - [x] Delete
- [x] Add logs for writing and reading (log to console and also save in a database table containing action, origin, timestamps, etc)
- [x] Add input validation on routes
- [x] Add jwt authentication on routes (generate API-KEY to validate requests)
- [x] Add custom error and success handlers
- [x] Add route documentation with swagger
- [x] Implement queue with RabbitMQ
- [x] Implement cache with Redis
- [x] Add unit tests
- [x] Add end to end tests
- [x] Create docker-compose configuration to start redis, rabbitmq and mongodb

### Frontend
- [x] Create initial setup
- [x] Create submit button component
- [x] Create text input component
- [x] Create table row component with delete and update button
- [x] Combine components to create listing table
- [x] Combine components to create creation form
- [x] Create backend connection
- [x] Add requests for routes
- [x] Add input validator in forms
- [x] Add error handler
- [x] Add filters and sorting in table
- [x] Add loading with skeleton

### Next Features
- [ ] Add ZIP code API request
- [ ] Add dark/light mode
- [ ] Add pt-BR / en-US translation
