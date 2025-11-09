# 🚀 Financial Transaction API with PyODBC, Pagination, Authentication, and Audit Logging

## 📌 Table of Contents

- [Introduction](#Introduction)
- [Project Setup](#project-setup)
- [Defining the Transaction Model](#defining-the-transaction-model)
- [Database Integration with PyODBC](#database-integration-with-pyodbc)
- [Implementing Pagination](#implementing-pagination)
- [JWT Authentication](#jwt-authentication)
- [Audit Logging for Compliance](#audit-logging-for-compliance)
- [Final Thoughts](#final-thoughts)
- [Back to TOC](#table-of-contents)

## 📖 Introduction

This guide provides a robust implementation of a **Financial Transaction API** using:

- **FastAPI** for high-performance API development.
- **PyODBC** to interact with a SQL Server database.
- **JWT Authentication** to secure access.
- **Audit Logging** for compliance with financial regulations.
- **Pagination** to optimize database queries and API responses.

## ⚙️ Project Setup

### 🔹 Install Dependencies

Run the following command to install the required packages:

```sh
pip install fastapi pyodbc passlib[bcrypt] python-jose[cryptography] fastapi-jwt-auth uvicorn
```

### 🔹 Database Configuration

Update the **SQL Server connection string**:

```python
DATABASE_CONFIG = {
    'server': 'your_sql_server',
    'database': 'your_database',
    'username': 'your_username',
    'password': 'your_password',
    'driver': '{ODBC Driver 17 for SQL Server}'
}
```

## 📊 Defining the Transaction Model

Define the **Transaction schema** using Pydantic:

```python
from pydantic import BaseModel, Field
from datetime import datetime
from uuid import UUID, uuid4

class Transaction(BaseModel):
    id: UUID = Field(default_factory=uuid4)
    amount: float
    currency: str
    transaction_type: str
    account_id: UUID
    timestamp: datetime = Field(default_factory=datetime.utcnow)
```

[🔝 Back to TOC](# 📌 table-of-contents)

## 🛢️ Database Integration with PyODBC

### 🔹 Establish Connection

```python
import pyodbc

def get_db_connection():
    conn_str = (
        f"DRIVER={DATABASE_CONFIG['driver']};"
        f"SERVER={DATABASE_CONFIG['server']};"
        f"DATABASE={DATABASE_CONFIG['database']};"
        f"UID={DATABASE_CONFIG['username']};"
        f"PWD={DATABASE_CONFIG['password']}"
    )
    return pyodbc.connect(conn_str)
```

### 🔹 Insert a Transaction

```python
def insert_transaction(transaction: Transaction):
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO transactions (id, amount, currency, transaction_type, account_id, timestamp) VALUES (?, ?, ?, ?, ?, ?)",
        transaction.id, transaction.amount, transaction.currency, transaction.transaction_type, transaction.account_id, transaction.timestamp
    )
    conn.commit()
    conn.close()
```

[🔝 Back to TOC](# 📌 table-of-contents)

## 📑 Implementing Pagination

```python
from fastapi import Query

def get_transactions(page: int = Query(1, ge=1), size: int = Query(10, le=100)):
    offset = (page - 1) * size
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute("SELECT * FROM transactions ORDER BY timestamp DESC OFFSET ? ROWS FETCH NEXT ? ROWS ONLY", (offset, size))
    transactions = cursor.fetchall()
    conn.close()
    return transactions
```

[🔝 Back to TOC](# 📌 table-of-contents)

## 🔐 JWT Authentication

### 🔹 Generate JWT Token

```python
from jose import jwt
from datetime import datetime, timedelta

SECRET_KEY = "supersecretkey"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    return jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
```

### 🔹 Protect API Endpoints

```python
from fastapi import Security, HTTPException, Depends
from fastapi.security import OAuth2PasswordBearer

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def authenticate_user(token: str = Security(oauth2_scheme)):
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        return payload["sub"]
    except:
        raise HTTPException(status_code=401, detail="Invalid token")
```

[🔝 Back to TOC](# 📌 table-of-contents)

## 📝 Audit Logging for Compliance

### 🔹 Create Audit Log Table

```python
def log_audit(user_id: str, action: str, transaction_id: UUID):
    conn = get_db_connection()
    cursor = conn.cursor()
    cursor.execute(
        "INSERT INTO audit_logs (user_id, action, transaction_id, timestamp) VALUES (?, ?, ?, ?)",
        user_id, action, transaction_id, datetime.utcnow()
    )
    conn.commit()
    conn.close()
```

### 🔹 Log Transaction Events

```python
async def create_transaction(transaction: Transaction, token: str = Security(oauth2_scheme)):
    user_id = authenticate_user(token)
    insert_transaction(transaction)
    log_audit(user_id, "CREATE_TRANSACTION", transaction.id)
    return transaction
```

[🔝 Back to TOC](# 📌 table-of-contents)

## 🎯 Final Thoughts

This **FastAPI + PyODBC** implementation ensures:

- ✅ **Secure Transactions** with JWT Authentication 🔐
- ✅ **Optimized Database Queries** using Pagination 📑
- ✅ **Financial Compliance** with Audit Logging 📊

Would you like additional **role-based authorization, rate limiting, or alerts for fraud detection?** 🚀

[🔝 Back to TOC](# 📌 table-of-contents)

