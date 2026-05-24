!![API Gateway Diagram|697](assets/images/api-gateway.png)



👉 **API Gateway is a single entry point that receives client requests and routes them to the appropriate backend services.**

👉 It also handles **security (Auth), rate limiting, and monitoring** before forwarding the request.

👉 In short, it acts as a **gateway between users and microservices, managing and protecting all API traffic.**

API Gateway:

- 🔐 **AuthN** → Checks who you are
- 🛡️ **AuthZ** → Checks what you can access
- 🔒 **SSL Termination** → Handles HTTPS security
- ⚡ **Rate Limiting** → Prevents overload
- 📊 **Monitoring** → Logs & tracks requests

# 🧠 What This Diagram Represents
👉 This diagram shows:

```text
Clients → API Gateway → Backend Services (Microservices)
```

✔️ API Gateway acts as a **single entry point** for all clients


# 👥 1. Clients (Left Side)

You have different types of users:

- 🌐 Web users (browser)
- 📱 Mobile users (apps)
    

```text
Users → Internet → API Gateway
```

👉 They don’t directly talk to backend services

---

# 🚪 2. API Gateway (Center – Most Important)

This is the **brain / control layer** of your system

Instead of calling services directly, all requests go through API Gateway.


## 🔐 What API Gateway Does 

### 🟢 1. AuthN (Authentication)

💡 “Who are you?”
- Verifies identity
- Example:
    - Login with token
    - Cognito / JWT validation
👉 Only valid users allowed

---

### 🔵 2. AuthZ (Authorization)

💡 “What are you allowed to do?”

- Checks permissions
    
- Example:
    
    - Admin vs normal user
        
    - Access control
        

👉 Even valid user can be restricted

---

### 🔒 3. SSL Termination

💡 Handles HTTPS security

- Decrypts incoming HTTPS request
    
- Sends plain request internally
    

👉 Improves performance & security

---

### ⚡ 4. Rate Limiting

💡 Protects your system from overload

- Limits requests per user/IP
    
- Example:
    
    - 100 requests/minute
        

👉 Prevents:

- DDoS attacks
    
- Abuse
    

---

### 📊 5. Monitoring

💡 Tracks everything

- Logs requests
- Measures latency
- Sends metrics to CloudWatch

👉 Helps debugging & scaling

---

### ➕ 6. etc.

Includes:
- Request validation
- Transformation
- Caching 
- API keys


# 🔀 3. Backend Services (Right Side)

These are **microservices**, each handling a specific function:

## 🧩 Services Explained

### 📦 Inventory Service

- Manages stock
- Example: “How many items available?”

### 🛍️ Product Info Service

- Product details
- Name, price, description

### 🛒 Cart Service

- User shopping cart
- Add/remove items

### ⭐ Reviews Service

- Ratings & reviews


# 🔄 Full Request Flow

```text
1. User sends request (mobile/web)
2. Request hits API Gateway
3. API Gateway:
   - Authenticates user
   - Checks permissions
   - Applies rate limit
   - Logs request
4. Routes request to correct service
5. Service processes request
6. Response goes back via API Gateway
7. User gets response
```

---

# 🎯 Why This Architecture is Used

## ❌ Without API Gateway

- Client talks to multiple services
    
- Complex & insecure
    

## ✅ With API Gateway

- Single entry point ✔️
    
- Centralized security ✔️
    
- Easier management ✔️
    

---

# 🧠 Easy Analogy

```text
API Gateway = Security Guard + Receptionist

Clients = Visitors  
Services = Offices inside building
```

👉 Guard checks:

- Who you are (AuthN)
    
- Where you can go (AuthZ)
    

👉 Receptionist routes you to correct department

---

# 🚀 Real AWS Mapping

|Diagram Component|AWS Service|
|---|---|
|API Gateway|AWS API Gateway|
|AuthN/AuthZ|Cognito / IAM / Lambda Authorizer|
|Monitoring|CloudWatch|
|Backend Services|Lambda / ECS / EC2 / ALB|

---

# 🎯 Final Takeaway

👉 API Gateway is:

```text
"Single secure entry point that manages, protects, and routes API requests"
```



![img](assets/images/rest-api-integration.png)


# 🔄 REST API Integration Flow (Your Diagram)

```text
Client → Method Request → Integration Request → Backend (Mock)
       ← Method Response ← Integration Response ←
```

---

# 🧩 Each Component Explained

---

## 👤 1. Client

- User / app sends request  
    👉 Example: `GET /users`
    


## 🟢 2. Method Request (Entry Gate 🚪)

💡 First step inside API Gateway

What happens here:

- Authentication (AuthN)
- Authorization (AuthZ)
- Validate request (params, headers, body)

👉 If invalid → request rejected ❌


## 🔵 3. Integration Request

💡 Prepares request for backend

- Transforms request (mapping templates)
- Adds/changes headers or body
- Decides where to send request

👉 Converts **client format → backend format**


## 🟡 4. Integration (Backend)

In your diagram: **Mock Integration**

💡 Instead of real backend:

- API Gateway returns fixed response

👉 Used for:

- Testing
- Demo
- Prototyping


## 🟣 5. Integration Response

💡 Backend response comes back here

- Transform response
- Map status codes
- Modify data if needed

👉 Converts **backend format → API format**

---

## 🟠 6. Method Response (Final Output)

💡 Final response sent to client

- Defines response structure
    
- Status codes (200, 400, etc.)
    
- Headers
    

👉 What client finally receives

---

# 🔄 Full Flow in Simple Words

```text
1. Client sends request
2. Method Request → validate & authorize
3. Integration Request → prepare for backend
4. Backend (Mock) → generate response
5. Integration Response → transform response
6. Method Response → send final response to client
```

---

# 🧠 Easy Way to Remember

```text
Request Path:
Client → Validate → Transform → Backend

Response Path:
Backend → Transform → Send to Client
```

---

# 🎯 Key Insight

👉 REST API gives **full control at each step**:

- Before backend (Method + Integration Request)
    
- After backend (Integration + Method Response)
    

---

# 🚀 Real vs Your Diagram

|Component|Your Diagram|
|---|---|
|Backend|Mock (fake response)|
|Real world|Lambda / HTTP / DB|



## 🔴 Non-Proxy

Client sends:

{ "name": "John" }

You can:

- Modify request
- Filter fields
- Change response

## 🟢 Proxy

Client sends:

{ "name": "John" }

👉 Backend receives EXACT SAME data  
👉 Backend must handle everything

---

# 🧠 Easy Way to Remember

Non-Proxy = API Gateway controls everything    
Proxy     = Backend controls everything


# 🎯 What is method Request Validation?

👉 **Request Validation = API Gateway checks incoming request BEFORE sending to backend**

✔️ If request is valid → goes to backend  
❌ If invalid → rejected immediately (no backend call)

---

# 🧩 What Can Be Validated?

API Gateway can validate 3 things:

1. Query String Parameters
2. HTTP Headers
3. Request Body


# 🎯 What is a Query String Parameter?

👉 It is **data that the client attaches to the URL** while making a request

Query string parameters are key-value pairs added by the client in the URL after “?” and sent as part of the HTTP request to API Gateway. API Gateway extracts them and passes them to the backend for processing.

https://api.myapp.com/products?category=electronics&id=101

# 🎯 What is a http  header Parameter?

HTTP headers are key-value pairs sent with a request or response that provide metadata such as authentication, content type, and client information, helping the server understand how to process the request

```
Content-Type: application/json
```


# 🎯 What is a requets body?

Request body validation means API Gateway checks if the JSON data sent by the client is correct before sending it to the backend.

```
{
  "name": "John",
  "age": 25
}
```
