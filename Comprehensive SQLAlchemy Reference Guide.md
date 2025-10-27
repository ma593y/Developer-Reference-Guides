# Comprehensive SQLAlchemy Reference Guide

## Table of Contents
1. [Introduction & Architecture](#introduction--architecture)
2. [Installation & Setup](#installation--setup)
3. [Engine & Connection Management](#engine--connection-management)
4. [SQLAlchemy Core](#sqlalchemy-core)
5. [SQLAlchemy ORM](#sqlalchemy-orm)
6. [Declarative Mapping](#declarative-mapping)
7. [Column Types & Options](#column-types--options)
8. [Relationships](#relationships)
9. [Querying](#querying)
10. [Session Management](#session-management)
11. [Transactions](#transactions)
12. [Advanced ORM Patterns](#advanced-orm-patterns)
13. [Performance & Optimization](#performance--optimization)
14. [Migration with Alembic](#migration-with-alembic)

---

## 1. Introduction & Architecture

### What is SQLAlchemy?
SQLAlchemy is a Python SQL toolkit and Object-Relational Mapping (ORM) library that provides a full suite of enterprise-level persistence patterns.

### Two-Tier Architecture
- **SQLAlchemy Core**: SQL abstraction toolkit (lower level)
- **SQLAlchemy ORM**: Object-Relational Mapper built on Core (higher level)

### Key Components
- **Engine**: Connection pool and dialect manager
- **Connection**: Single database connection
- **Session**: ORM's "workspace" for objects
- **MetaData**: Collection of table definitions
- **Declarative Base**: Base class for ORM models

---

## 2. Installation & Setup

### Installation
```bash
# Basic installation
pip install sqlalchemy

# With specific database drivers
pip install sqlalchemy psycopg2-binary  # PostgreSQL
pip install sqlalchemy pymysql          # MySQL
pip install sqlalchemy cx_oracle        # Oracle
```

### Database Drivers
- **PostgreSQL**: `psycopg2`, `psycopg2-binary`, `asyncpg`
- **MySQL**: `pymysql`, `mysqlclient`, `aiomysql`
- **SQLite**: Built-in (no driver needed)
- **Oracle**: `cx_oracle`
- **MS SQL Server**: `pyodbc`, `pymssql`

---

## 3. Engine & Connection Management

### Creating an Engine

```python
from sqlalchemy import create_engine

# SQLite (file-based)
engine = create_engine('sqlite:///database.db')

# SQLite (in-memory)
engine = create_engine('sqlite:///:memory:')

# PostgreSQL
engine = create_engine('postgresql://user:password@localhost:5432/dbname')

# MySQL
engine = create_engine('mysql+pymysql://user:password@localhost:3306/dbname')

# Oracle
engine = create_engine('oracle+cx_oracle://user:password@localhost:1521/dbname')

# SQL Server
engine = create_engine('mssql+pyodbc://user:password@localhost/dbname')
```

### Engine Options

```python
engine = create_engine(
    'postgresql://user:pass@localhost/db',
    echo=True,                    # Log SQL statements
    pool_size=5,                  # Connection pool size
    max_overflow=10,              # Max connections beyond pool_size
    pool_timeout=30,              # Timeout for getting connection
    pool_recycle=3600,           # Recycle connections after 1 hour
    pool_pre_ping=True,          # Test connections before using
    connect_args={'timeout': 15} # Driver-specific arguments
)
```

### Connection Patterns

```python
# Using context manager (recommended)
with engine.connect() as conn:
    result = conn.execute(text("SELECT * FROM users"))
    
# Begin transaction explicitly
with engine.begin() as conn:
    conn.execute(text("INSERT INTO users (name) VALUES ('Alice')"))
    # Auto-commits on exit

# Traditional pattern
conn = engine.connect()
try:
    result = conn.execute(text("SELECT * FROM users"))
finally:
    conn.close()
```

---

## 4. SQLAlchemy Core

### MetaData and Table Definition

```python
from sqlalchemy import MetaData, Table, Column, Integer, String, ForeignKey

metadata = MetaData()

users = Table(
    'users',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('name', String(50), nullable=False),
    Column('email', String(100), unique=True),
    Column('age', Integer)
)

addresses = Table(
    'addresses',
    metadata,
    Column('id', Integer, primary_key=True),
    Column('user_id', Integer, ForeignKey('users.id')),
    Column('email_address', String(100), nullable=False)
)
```

### Creating Tables

```python
# Create all tables
metadata.create_all(engine)

# Create specific table
users.create(engine)

# Drop tables
metadata.drop_all(engine)

# Check if table exists
users.exists(engine)
```

### Reflecting Existing Tables

```python
# Reflect entire database
metadata = MetaData()
metadata.reflect(bind=engine)
users = metadata.tables['users']

# Reflect single table
users = Table('users', metadata, autoload_with=engine)
```

### Core SQL Expressions

#### INSERT Operations
```python
from sqlalchemy import insert

# Single insert
stmt = insert(users).values(name='Alice', email='alice@example.com')
conn.execute(stmt)

# Multiple inserts
stmt = insert(users)
conn.execute(stmt, [
    {'name': 'Bob', 'email': 'bob@example.com'},
    {'name': 'Charlie', 'email': 'charlie@example.com'}
])

# Insert with RETURNING (PostgreSQL)
stmt = insert(users).values(name='Dave').returning(users.c.id)
result = conn.execute(stmt)
new_id = result.scalar()
```

#### SELECT Operations
```python
from sqlalchemy import select

# Simple select
stmt = select(users)
result = conn.execute(stmt)

# Select specific columns
stmt = select(users.c.name, users.c.email)

# With WHERE clause
stmt = select(users).where(users.c.age > 25)

# Multiple conditions
stmt = select(users).where(
    (users.c.age > 25) & (users.c.name.like('A%'))
)

# OR conditions
stmt = select(users).where(
    (users.c.age < 20) | (users.c.age > 60)
)

# IN clause
stmt = select(users).where(users.c.name.in_(['Alice', 'Bob', 'Charlie']))

# ORDER BY
stmt = select(users).order_by(users.c.name.desc())

# LIMIT and OFFSET
stmt = select(users).limit(10).offset(20)

# DISTINCT
stmt = select(users.c.name).distinct()
```

#### UPDATE Operations
```python
from sqlalchemy import update

# Update with WHERE
stmt = update(users).where(users.c.name == 'Alice').values(age=30)
conn.execute(stmt)

# Update multiple columns
stmt = update(users).where(users.c.id == 1).values(
    name='Alice Updated',
    email='alice_new@example.com'
)

# Conditional update
stmt = update(users).where(users.c.age < 18).values(age=18)
```

#### DELETE Operations
```python
from sqlalchemy import delete

# Delete with WHERE
stmt = delete(users).where(users.c.name == 'Alice')
conn.execute(stmt)

# Delete with multiple conditions
stmt = delete(users).where(
    (users.c.age < 18) & (users.c.active == False)
)
```

### Joins

```python
# INNER JOIN
stmt = select(users, addresses).join(
    addresses, users.c.id == addresses.c.user_id
)

# Alternative join syntax
stmt = select(users, addresses).join_from(
    users, addresses, users.c.id == addresses.c.user_id
)

# LEFT OUTER JOIN
stmt = select(users, addresses).outerjoin(
    addresses, users.c.id == addresses.c.user_id
)

# Multiple joins
stmt = select(users).join(addresses).join(orders)

# Join with WHERE
stmt = select(users, addresses).join(
    addresses
).where(users.c.age > 25)
```

### Aggregate Functions

```python
from sqlalchemy import func

# COUNT
stmt = select(func.count(users.c.id))

# COUNT with alias
stmt = select(func.count(users.c.id).label('user_count'))

# SUM, AVG, MIN, MAX
stmt = select(
    func.sum(orders.c.amount),
    func.avg(orders.c.amount),
    func.min(orders.c.amount),
    func.max(orders.c.amount)
)

# GROUP BY
stmt = select(
    users.c.age,
    func.count(users.c.id).label('count')
).group_by(users.c.age)

# HAVING
stmt = select(
    users.c.age,
    func.count(users.c.id).label('count')
).group_by(users.c.age).having(func.count(users.c.id) > 5)
```

### Subqueries

```python
# Scalar subquery
subq = select(func.avg(users.c.age)).scalar_subquery()
stmt = select(users).where(users.c.age > subq)

# Table subquery
subq = select(users.c.id).where(users.c.age > 25).subquery()
stmt = select(addresses).where(addresses.c.user_id.in_(select(subq)))

# CTE (Common Table Expression)
cte = select(users.c.id, users.c.name).where(users.c.age > 25).cte()
stmt = select(cte).where(cte.c.name.like('A%'))
```

---

## 5. SQLAlchemy ORM

### Creating a Session

```python
from sqlalchemy.orm import sessionmaker, Session

# Method 1: Using sessionmaker
SessionLocal = sessionmaker(bind=engine)
session = SessionLocal()

# Method 2: Direct Session
session = Session(engine)

# Method 3: Context manager (SQLAlchemy 1.4+)
with Session(engine) as session:
    # Work with session
    session.commit()
```

### Session Configuration

```python
SessionLocal = sessionmaker(
    bind=engine,
    autoflush=True,        # Auto-flush before queries
    autocommit=False,      # Don't auto-commit (recommended)
    expire_on_commit=True  # Expire objects after commit
)
```

---

## 6. Declarative Mapping

### Basic Model Definition

```python
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from typing import Optional

# SQLAlchemy 2.0 style
class Base(DeclarativeBase):
    pass

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str] = mapped_column(String(50))
    email: Mapped[str] = mapped_column(String(100), unique=True)
    age: Mapped[Optional[int]]  # Nullable column
    
    def __repr__(self):
        return f"<User(name={self.name}, email={self.email})>"
```

### Legacy Declarative Style

```python
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy import Column, Integer, String

Base = declarative_base()

class User(Base):
    __tablename__ = 'users'
    
    id = Column(Integer, primary_key=True)
    name = Column(String(50), nullable=False)
    email = Column(String(100), unique=True)
    age = Column(Integer)
```

### Table Arguments

```python
class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    email: Mapped[str]
    
    __table_args__ = (
        Index('idx_name_email', 'name', 'email'),
        UniqueConstraint('email', name='uq_email'),
        CheckConstraint('age >= 18', name='check_adult'),
        {'mysql_engine': 'InnoDB', 'schema': 'public'}
    )
```

---

## 7. Column Types & Options

### Common Column Types

```python
from sqlalchemy import (
    Integer, BigInteger, SmallInteger,
    String, Text, Unicode, UnicodeText,
    Float, Numeric, Boolean,
    Date, DateTime, Time, Interval,
    LargeBinary, Enum, JSON, ARRAY
)

class Example(Base):
    __tablename__ = 'examples'
    
    # Numeric types
    id: Mapped[int] = mapped_column(Integer, primary_key=True)
    big_num: Mapped[int] = mapped_column(BigInteger)
    small_num: Mapped[int] = mapped_column(SmallInteger)
    decimal_val: Mapped[Decimal] = mapped_column(Numeric(10, 2))
    float_val: Mapped[float] = mapped_column(Float)
    
    # String types
    name: Mapped[str] = mapped_column(String(100))
    description: Mapped[str] = mapped_column(Text)
    unicode_name: Mapped[str] = mapped_column(Unicode(100))
    
    # Boolean
    is_active: Mapped[bool] = mapped_column(Boolean, default=True)
    
    # Date/Time types
    created_at: Mapped[datetime] = mapped_column(DateTime, default=func.now())
    birth_date: Mapped[date] = mapped_column(Date)
    login_time: Mapped[time] = mapped_column(Time)
    
    # Binary
    file_data: Mapped[bytes] = mapped_column(LargeBinary)
    
    # JSON (PostgreSQL, MySQL 5.7+, SQLite 3.9+)
    metadata: Mapped[dict] = mapped_column(JSON)
    
    # Array (PostgreSQL)
    tags: Mapped[list] = mapped_column(ARRAY(String))
    
    # Enum
    status: Mapped[str] = mapped_column(
        Enum('active', 'inactive', 'pending', name='status_enum')
    )
```

### Column Options

```python
class User(Base):
    __tablename__ = 'users'
    
    # Primary key
    id: Mapped[int] = mapped_column(primary_key=True)
    
    # Auto-increment (implicit with Integer primary key)
    auto_id: Mapped[int] = mapped_column(autoincrement=True)
    
    # Nullable/Required
    required_field: Mapped[str] = mapped_column(nullable=False)
    optional_field: Mapped[Optional[str]]  # Nullable
    
    # Unique constraint
    email: Mapped[str] = mapped_column(unique=True)
    
    # Default values
    created_at: Mapped[datetime] = mapped_column(default=datetime.utcnow)
    status: Mapped[str] = mapped_column(default='active')
    
    # Server-side default
    server_default: Mapped[str] = mapped_column(server_default='default_value')
    
    # Index
    indexed_field: Mapped[str] = mapped_column(index=True)
    
    # Column name different from attribute name
    user_name: Mapped[str] = mapped_column('username', String(50))
    
    # Onupdate (for timestamps)
    updated_at: Mapped[datetime] = mapped_column(
        default=func.now(),
        onupdate=func.now()
    )
    
    # Comment
    description: Mapped[str] = mapped_column(comment='User description')
```

### Custom Types

```python
from sqlalchemy.types import TypeDecorator

class PasswordType(TypeDecorator):
    impl = String
    cache_ok = True
    
    def process_bind_param(self, value, dialect):
        # Hash password before storing
        if value is not None:
            return hash_password(value)
        return value
    
    def process_result_value(self, value, dialect):
        # Return hashed value as-is
        return value

class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    password: Mapped[str] = mapped_column(PasswordType(128))
```

---

## 8. Relationships

### One-to-Many Relationship

```python
from sqlalchemy.orm import relationship

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    # One-to-Many: User has many addresses
    addresses: Mapped[list["Address"]] = relationship(back_populates="user")

class Address(Base):
    __tablename__ = 'addresses'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    email_address: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey('users.id'))
    
    # Many-to-One: Address belongs to one user
    user: Mapped["User"] = relationship(back_populates="addresses")
```

### One-to-One Relationship

```python
class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    # One-to-One: uselist=False makes it singular
    profile: Mapped["Profile"] = relationship(
        back_populates="user",
        uselist=False
    )

class Profile(Base):
    __tablename__ = 'profiles'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    bio: Mapped[str]
    user_id: Mapped[int] = mapped_column(ForeignKey('users.id'), unique=True)
    
    user: Mapped["User"] = relationship(back_populates="profile")
```

### Many-to-Many Relationship

```python
# Association table
user_group_association = Table(
    'user_group',
    Base.metadata,
    Column('user_id', Integer, ForeignKey('users.id')),
    Column('group_id', Integer, ForeignKey('groups.id'))
)

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    # Many-to-Many
    groups: Mapped[list["Group"]] = relationship(
        secondary=user_group_association,
        back_populates="users"
    )

class Group(Base):
    __tablename__ = 'groups'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    users: Mapped[list["User"]] = relationship(
        secondary=user_group_association,
        back_populates="groups"
    )
```

### Many-to-Many with Association Object

```python
class UserGroup(Base):
    __tablename__ = 'user_group'
    
    user_id: Mapped[int] = mapped_column(ForeignKey('users.id'), primary_key=True)
    group_id: Mapped[int] = mapped_column(ForeignKey('groups.id'), primary_key=True)
    role: Mapped[str]  # Additional data
    joined_at: Mapped[datetime] = mapped_column(default=func.now())
    
    # Relationships to parent tables
    user: Mapped["User"] = relationship(back_populates="group_associations")
    group: Mapped["Group"] = relationship(back_populates="user_associations")

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    group_associations: Mapped[list["UserGroup"]] = relationship(
        back_populates="user"
    )
    
    # Convenience property to access groups directly
    @property
    def groups(self):
        return [assoc.group for assoc in self.group_associations]

class Group(Base):
    __tablename__ = 'groups'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    user_associations: Mapped[list["UserGroup"]] = relationship(
        back_populates="group"
    )
```

### Relationship Options

```python
class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    
    addresses: Mapped[list["Address"]] = relationship(
        back_populates="user",
        
        # Cascade options
        cascade="all, delete-orphan",  # Delete addresses when user deleted
        
        # Lazy loading strategy
        lazy="select",  # 'select', 'joined', 'subquery', 'dynamic', 'raise'
        
        # Order results
        order_by="Address.email_address",
        
        # Filter results
        primaryjoin="and_(User.id==Address.user_id, Address.active==True)",
        
        # Foreign keys (explicit)
        foreign_keys="Address.user_id",
        
        # Specify remote side in self-referential relationship
        remote_side="User.id",
        
        # Join condition for complex relationships
        secondary=user_group_association,
        
        # Viewonly (no changes tracked)
        viewonly=True,
        
        # Collection class
        collection_class=set  # Use set instead of list
    )
```

### Self-Referential Relationship

```python
class Employee(Base):
    __tablename__ = 'employees'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    manager_id: Mapped[Optional[int]] = mapped_column(ForeignKey('employees.id'))
    
    # Self-referential relationships
    manager: Mapped[Optional["Employee"]] = relationship(
        back_populates="subordinates",
        remote_side=[id]  # Specify which side is "remote"
    )
    
    subordinates: Mapped[list["Employee"]] = relationship(
        back_populates="manager"
    )
```

### Cascade Options

```python
# Common cascade patterns:

# Delete children when parent is deleted
cascade="all, delete"

# Delete orphaned children (removed from relationship)
cascade="all, delete-orphan"

# Only save operations cascade
cascade="save-update"

# Merge operations cascade
cascade="merge"

# Refresh operations cascade
cascade="refresh"

# Expunge operations cascade
cascade="expunge"

# No cascading
cascade=None

# Example usage
class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    
    # Children deleted when parent deleted AND when removed from collection
    addresses: Mapped[list["Address"]] = relationship(
        cascade="all, delete-orphan"
    )
```

### Loading Strategies

```python
from sqlalchemy.orm import selectinload, joinedload, subqueryload

# Lazy loading (default) - query per relationship access
addresses: Mapped[list["Address"]] = relationship(lazy="select")

# Joined loading - JOIN in single query
addresses: Mapped[list["Address"]] = relationship(lazy="joined")

# Subquery loading - second query with IN clause
addresses: Mapped[list["Address"]] = relationship(lazy="subquery")

# Select in loading - separate SELECT with IN (recommended for collections)
addresses: Mapped[list["Address"]] = relationship(lazy="selectin")

# Raise error if accessed (prevent N+1)
addresses: Mapped[list["Address"]] = relationship(lazy="raise")

# Dynamic - return Query object instead of loading
addresses = relationship(lazy="dynamic")

# No automatic loading
addresses: Mapped[list["Address"]] = relationship(lazy="noload")
```

---

## 9. Querying

### Basic ORM Queries

```python
from sqlalchemy import select

# Get all records
stmt = select(User)
users = session.execute(stmt).scalars().all()

# Or using legacy query API
users = session.query(User).all()

# Get first record
user = session.execute(select(User)).scalars().first()

# Get one record (raises if not found or multiple found)
user = session.execute(select(User).where(User.id == 1)).scalar_one()

# Get one or none
user = session.execute(select(User).where(User.id == 1)).scalar_one_or_none()

# Get by primary key
user = session.get(User, 1)  # Fast, checks identity map first
```

### Filtering

```python
# WHERE conditions
stmt = select(User).where(User.name == 'Alice')
stmt = select(User).where(User.age > 25)
stmt = select(User).where(User.email.like('%@example.com'))
stmt = select(User).where(User.name.in_(['Alice', 'Bob']))
stmt = select(User).where(User.age.between(18, 65))
stmt = select(User).where(User.email.is_(None))
stmt = select(User).where(User.email.is_not(None))

# Multiple conditions (AND)
stmt = select(User).where(User.age > 25, User.name.like('A%'))
stmt = select(User).where(User.age > 25).where(User.name.like('A%'))

# OR conditions
from sqlalchemy import or_
stmt = select(User).where(or_(User.age < 18, User.age > 65))

# AND conditions (explicit)
from sqlalchemy import and_
stmt = select(User).where(and_(User.age > 18, User.email.is_not(None)))

# NOT conditions
from sqlalchemy import not_
stmt = select(User).where(not_(User.name.like('A%')))

# Complex conditions
stmt = select(User).where(
    or_(
        and_(User.age > 18, User.age < 25),
        and_(User.age > 60, User.status == 'retired')
    )
)
```

### String Operations

```python
# LIKE (case-sensitive in most databases)
stmt = select(User).where(User.name.like('A%'))
stmt = select(User).where(User.name.like('%alice%'))

# ILIKE (case-insensitive, PostgreSQL)
stmt = select(User).where(User.name.ilike('%alice%'))

# Starts with
stmt = select(User).where(User.name.startswith('A'))

# Ends with
stmt = select(User).where(User.name.endswith('son'))

# Contains
stmt = select(User).where(User.name.contains('ali'))

# Regex match (PostgreSQL)
stmt = select(User).where(User.name.op('~')('A.*'))
```

### Ordering

```python
# Ascending order
stmt = select(User).order_by(User.name)
stmt = select(User).order_by(User.name.asc())

# Descending order
stmt = select(User).order_by(User.age.desc())

# Multiple columns
stmt = select(User).order_by(User.age.desc(), User.name.asc())

# Nulls first/last
stmt = select(User).order_by(User.age.desc().nullsfirst())
stmt = select(User).order_by(User.age.asc().nullslast())
```

### Limiting and Pagination

```python
# LIMIT
stmt = select(User).limit(10)

# OFFSET
stmt = select(User).offset(20)

# Pagination
page = 2
page_size = 10
stmt = select(User).limit(page_size).offset((page - 1) * page_size)

# Slice notation (legacy Query API)
users = session.query(User)[10:20]  # OFFSET 10 LIMIT 10
```

### Aggregation

```python
from sqlalchemy import func

# COUNT
stmt = select(func.count(User.id))
count = session.execute(stmt).scalar()

# Or count all
stmt = select(func.count()).select_from(User)

# SUM, AVG, MIN, MAX
stmt = select(
    func.sum(Order.amount),
    func.avg(Order.amount),
    func.min(Order.amount),
    func.max(Order.amount)
)

# GROUP BY
stmt = select(
    User.age,
    func.count(User.id).label('count')
).group_by(User.age)

# HAVING
stmt = select(
    User.age,
    func.count(User.id).label('count')
).group_by(User.age).having(func.count(User.id) > 5)
```

### Joins

```python
# Implicit join (via relationship)
stmt = select(User).join(User.addresses)

# Explicit join
stmt = select(User).join(Address, User.id == Address.user_id)

# Outer join
stmt = select(User).outerjoin(Address)

# Multiple joins
stmt = select(User).join(Address).join(Order)

# Filter on joined table
stmt = select(User).join(Address).where(Address.email.like('%@example.com'))

# Select from both tables
stmt = select(User, Address).join(Address)
results = session.execute(stmt).all()
for user, address in results:
    print(user.name, address.email)
```

### Eager Loading

```python
from sqlalchemy.orm import selectinload, joinedload, subqueryload

# Select in load (recommended for collections)
stmt = select(User).options(selectinload(User.addresses))

# Joined load (single query with JOIN)
stmt = select(User).options(joinedload(User.addresses))

# Subquery load
stmt = select(User).options(subqueryload(User.addresses))

# Nested loading
stmt = select(User).options(
    selectinload(User.addresses).selectinload(Address.orders)
)

# Load only specific columns from related object
from sqlalchemy.orm import load_only
stmt = select(User).options(
    selectinload(User.addresses).load_only(Address.email)
)
```

### Subqueries

```python
# Scalar subquery
avg_age = select(func.avg(User.age)).scalar_subquery()
stmt = select(User).where(User.age > avg_age)

# EXISTS
subq = select(Address.id).where(Address.user_id == User.id).exists()
stmt = select(User).where(subq)

# NOT EXISTS
stmt = select(User).where(~subq)

# IN with subquery
subq = select(Address.user_id).where(Address.email.like('%@example.com'))
stmt = select(User).where(User.id.in_(subq))

# Lateral subquery (PostgreSQL)
subq = select(Address).where(Address.user_id == User.id).lateral()
stmt = select(User, subq).join(subq)
```

### Common Table Expressions (CTE)

```python
# Simple CTE
cte = select(User.id, User.name).where(User.age > 25).cte()
stmt = select(cte).where(cte.c.name.like('A%'))

# Recursive CTE
from sqlalchemy import union_all

# Anchor member
anchor = select(Employee.id, Employee.name, Employee.manager_id).where(
    Employee.manager_id.is_(None)
)

# Recursive member
cte = anchor.cte(name="employee_hierarchy", recursive=True)
recursive = select(Employee.id, Employee.name, Employee.manager_id).join(
    cte, Employee.manager_id == cte.c.id
)

# Union
cte = cte.union_all(recursive)
stmt = select(cte)
```

### Window Functions

```python
from sqlalchemy import over

# ROW_NUMBER
stmt = select(
    User.id,
    User.name,
    func.row_number().over(order_by=User.name).label('row_num')
)

# RANK and DENSE_RANK
stmt = select(
    User.name,
    User.age,
    func.rank().over(order_by=User.age.desc()).label('age_rank')
)

# PARTITION BY
stmt = select(
    User.department,
    User.salary,
    func.avg(User.salary).over(partition_by=User.department).label('dept_avg')
)

# LEAD and LAG
stmt = select(
    User.name,
    User.created_at,
    func.lag(User.created_at).over(order_by=User.created_at).label('prev_created')
)
```

### Raw SQL

```python
from sqlalchemy import text

# Execute raw SQL
result = session.execute(text("SELECT * FROM users WHERE age > :age"), {"age": 25})

# With ORM objects
stmt = text("SELECT * FROM users WHERE age > :age")
result = session.execute(stmt, {"age": 25})
users = result.scalars(User).all()

# TextClause with columns
stmt = text("SELECT id, name, age FROM users").columns(User.id, User.name, User.age)
result = session.execute(stmt)
```

---

## 10. Session Management

### Adding Objects

```python
# Add single object
user = User(name='Alice', email='alice@example.com')
session.add(user)
session.commit()

# Add multiple objects
users = [
    User(name='Bob', email='bob@example.com'),
    User(name='Charlie', email='charlie@example.com')
]
session.add_all(users)
session.commit()

# Object states after add
# - Transient: not in session, no database identity
# - Pending: in session, not yet flushed to database
# - Persistent: in session, has database identity
# - Detached: has database identity, not in session
# - Deleted: marked for deletion
```

### Updating Objects

```python
# Method 1: Fetch and modify
user = session.get(User, 1)
user.name = 'Alice Updated'
session.commit()  # Auto-detected change

# Method 2: Bulk update
stmt = update(User).where(User.age < 18).values(age=18)
session.execute(stmt)
session.commit()

# Method 3: Update with returning
stmt = update(User).where(User.id == 1).values(
    name='Updated'
).returning(User)
updated_user = session.execute(stmt).scalar_one()
session.commit()
```

### Deleting Objects

```python
# Method 1: Delete instance
user = session.get(User, 1)
session.delete(user)
session.commit()

# Method 2: Bulk delete
stmt = delete(User).where(User.age < 18)
session.execute(stmt)
session.commit()
```

### Flushing

```python
# Flush manually (sync session with database without committing)
user = User(name='Alice')
session.add(user)
session.flush()  # INSERT executed, user.id populated
# Transaction not committed yet

# Auto-flush (default behavior)
session.add(user)
# Flush happens automatically before queries
result = session.execute(select(User))  # Auto-flush before query

# Disable auto-flush temporarily
with session.no_autoflush:
    # Queries won't trigger auto-flush
    pass
```

### Committing and Rolling Back

```python
# Commit transaction
session.commit()

# Rollback transaction
try:
    user = User(name='Alice')
    session.add(user)
    session.commit()
except Exception:
    session.rollback()
    raise

# Nested transactions (savepoints)
session.begin_nested()
try:
    user = User(name='Alice')
    session.add(user)
    session.commit()  # Commits savepoint
except Exception:
    session.rollback()  # Rolls back to savepoint
```

### Refreshing Objects

```python
# Refresh single object
user = session.get(User, 1)
session.refresh(user)  # Reload from database

# Refresh with specific attributes
session.refresh(user, ['name', 'email'])

# Expire object (mark for refresh on next access)
session.expire(user)
print(user.name)  # Triggers SELECT to reload

# Expire all objects in session
session.expire_all()
```

### Merging Objects

```python
# Merge detached object back into session
detached_user = User(id=1, name='Alice')
merged_user = session.merge(detached_user)
session.commit()

# Merge with load=False (don't query database)
merged_user = session.merge(detached_user, load=False)
```

### Expunging Objects

```python
# Remove object from session
user = session.get(User, 1)
session.expunge(user)  # Now detached

# Expunge all objects
session.expunge_all()
```

### Session States

```python
from sqlalchemy.orm import object_session
from sqlalchemy import inspect

# Check if object is in session
session = object_session(user)

# Check object state
insp = inspect(user)
print(insp.transient)   # True if never added to session
print(insp.pending)     # True if added but not flushed
print(insp.persistent)  # True if has database identity
print(insp.detached)    # True if was persistent but removed from session
print(insp.deleted)     # True if marked for deletion

# Check if object has uncommitted changes
print(insp.modified)
```

---

## 11. Transactions

### Basic Transaction Pattern

```python
# Implicit transaction (recommended)
with Session(engine) as session:
    user = User(name='Alice')
    session.add(user)
    session.commit()
    # Auto-rollback on exception

# Explicit transaction
session = Session(engine)
try:
    user = User(name='Alice')
    session.add(user)
    session.commit()
except Exception:
    session.rollback()
    raise
finally:
    session.close()
```

### Nested Transactions (Savepoints)

```python
session.begin()  # Start outer transaction

session.begin_nested()  # Savepoint
try:
    user1 = User(name='Alice')
    session.add(user1)
    session.commit()  # Commit savepoint
except Exception:
    session.rollback()  # Rollback to savepoint

session.commit()  # Commit outer transaction
```

### Two-Phase Commit

```python
# Prepare transaction
session.prepare()

# Commit prepared transaction
session.commit()

# Or rollback
session.rollback()
```

### Isolation Levels

```python
from sqlalchemy import create_engine

# Set isolation level on engine
engine = create_engine(
    'postgresql://user:pass@localhost/db',
    isolation_level="REPEATABLE READ"
)

# Isolation levels:
# - READ UNCOMMITTED
# - READ COMMITTED (default for most databases)
# - REPEATABLE READ
# - SERIALIZABLE

# Set isolation level per connection
with engine.connect().execution_options(
    isolation_level="SERIALIZABLE"
) as conn:
    # Use connection
    pass

# Set isolation level per session
session = Session(engine, isolation_level="REPEATABLE READ")
```

### Locking

```python
# SELECT FOR UPDATE (pessimistic locking)
stmt = select(User).where(User.id == 1).with_for_update()
user = session.execute(stmt).scalar_one()
user.balance -= 100
session.commit()

# SELECT FOR UPDATE NOWAIT
stmt = select(User).where(User.id == 1).with_for_update(nowait=True)

# SELECT FOR UPDATE SKIP LOCKED
stmt = select(User).where(User.id == 1).with_for_update(skip_locked=True)

# SELECT FOR SHARE (shared lock)
stmt = select(User).where(User.id == 1).with_for_update(read=True)

# Optimistic locking with version column
class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    version: Mapped[int] = mapped_column(default=0)
    
    __mapper_args__ = {"version_id_col": version}

# Update will fail if version changed
user = session.get(User, 1)
user.name = 'Updated'
session.commit()  # Raises StaleDataError if version mismatch
```

### Transaction Events

```python
from sqlalchemy import event

@event.listens_for(Session, "after_transaction_create")
def after_transaction_create(session, transaction):
    print("Transaction created")

@event.listens_for(Session, "after_transaction_end")
def after_transaction_end(session, transaction):
    print("Transaction ended")

@event.listens_for(Session, "after_commit")
def after_commit(session):
    print("Changes committed")

@event.listens_for(Session, "after_rollback")
def after_rollback(session):
    print("Changes rolled back")
```

---

## 12. Advanced ORM Patterns

### Mixins

```python
from datetime import datetime
from sqlalchemy.orm import declared_attr

class TimestampMixin:
    created_at: Mapped[datetime] = mapped_column(default=func.now())
    updated_at: Mapped[datetime] = mapped_column(
        default=func.now(),
        onupdate=func.now()
    )

class SoftDeleteMixin:
    deleted_at: Mapped[Optional[datetime]]
    
    @declared_attr
    def is_deleted(cls):
        return column_property(cls.deleted_at.is_not(None))

class User(Base, TimestampMixin, SoftDeleteMixin):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
```

### Abstract Base Classes

```python
class BaseModel(Base):
    __abstract__ = True
    
    id: Mapped[int] = mapped_column(primary_key=True)
    created_at: Mapped[datetime] = mapped_column(default=func.now())
    updated_at: Mapped[datetime] = mapped_column(
        default=func.now(),
        onupdate=func.now()
    )

class User(BaseModel):
    __tablename__ = 'users'
    name: Mapped[str]

class Product(BaseModel):
    __tablename__ = 'products'
    title: Mapped[str]
```

### Hybrid Properties

```python
from sqlalchemy.ext.hybrid import hybrid_property, hybrid_method

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    first_name: Mapped[str]
    last_name: Mapped[str]
    
    @hybrid_property
    def full_name(self):
        return f"{self.first_name} {self.last_name}"
    
    @full_name.expression
    def full_name(cls):
        return cls.first_name + " " + cls.last_name
    
    @hybrid_method
    def name_matches(self, pattern):
        return self.full_name.contains(pattern)
    
    @name_matches.expression
    def name_matches(cls, pattern):
        return cls.full_name.like(f"%{pattern}%")

# Usage
user = session.get(User, 1)
print(user.full_name)  # Instance level

# Query level
stmt = select(User).where(User.full_name == "John Doe")
stmt = select(User).where(User.name_matches("John"))
```

### Column Properties

```python
from sqlalchemy.orm import column_property

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    first_name: Mapped[str]
    last_name: Mapped[str]
    
    # SQL expression as column
    full_name = column_property(first_name + " " + last_name)
    
    # With subquery
    address_count = column_property(
        select(func.count(Address.id))
        .where(Address.user_id == id)
        .correlate_except(Address)
        .scalar_subquery()
    )
```

### Association Proxy

```python
from sqlalchemy.ext.associationproxy import association_proxy

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    
    # Regular relationship
    addresses: Mapped[list["Address"]] = relationship()
    
    # Proxy to access email_address directly
    emails = association_proxy('addresses', 'email_address')

class Address(Base):
    __tablename__ = 'addresses'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    user_id: Mapped[int] = mapped_column(ForeignKey('users.id'))
    email_address: Mapped[str]

# Usage
user = session.get(User, 1)
print(user.emails)  # ['alice@example.com', 'alice@work.com']
user.emails.append('alice@new.com')  # Creates new Address
```

### Polymorphic Inheritance

#### Joined Table Inheritance
```python
class Employee(Base):
    __tablename__ = 'employees'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    type: Mapped[str]
    
    __mapper_args__ = {
        'polymorphic_on': type,
        'polymorphic_identity': 'employee'
    }

class Engineer(Employee):
    __tablename__ = 'engineers'
    
    id: Mapped[int] = mapped_column(ForeignKey('employees.id'), primary_key=True)
    engineer_name: Mapped[str]
    
    __mapper_args__ = {
        'polymorphic_identity': 'engineer'
    }

class Manager(Employee):
    __tablename__ = 'managers'
    
    id: Mapped[int] = mapped_column(ForeignKey('employees.id'), primary_key=True)
    manager_name: Mapped[str]
    
    __mapper_args__ = {
        'polymorphic_identity': 'manager'
    }
```

#### Single Table Inheritance
```python
class Employee(Base):
    __tablename__ = 'employees'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    name: Mapped[str]
    type: Mapped[str]
    
    # Columns for subclasses
    engineer_info: Mapped[Optional[str]]
    manager_info: Mapped[Optional[str]]
    
    __mapper_args__ = {
        'polymorphic_on': type,
        'polymorphic_identity': 'employee'
    }

class Engineer(Employee):
    __mapper_args__ = {
        'polymorphic_identity': 'engineer'
    }

class Manager(Employee):
    __mapper_args__ = {
        'polymorphic_identity': 'manager'
    }
```

### Composite Types

```python
from sqlalchemy.orm import composite

class Point:
    def __init__(self, x, y):
        self.x = x
        self.y = y
    
    def __composite_values__(self):
        return self.x, self.y
    
    def __eq__(self, other):
        return self.x == other.x and self.y == other.y

class Vertex(Base):
    __tablename__ = 'vertices'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    x: Mapped[int]
    y: Mapped[int]
    
    point = composite(Point, x, y)

# Usage
vertex = Vertex(point=Point(3, 4))
print(vertex.point.x, vertex.point.y)
```

### Events and Listeners

```python
from sqlalchemy import event

class User(Base):
    __tablename__ = 'users'
    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str]

# Before insert
@event.listens_for(User, 'before_insert')
def before_insert(mapper, connection, target):
    target.email = target.email.lower()

# After insert
@event.listens_for(User, 'after_insert')
def after_insert(mapper, connection, target):
    print(f"User {target.id} inserted")

# Before update
@event.listens_for(User, 'before_update')
def before_update(mapper, connection, target):
    pass

# Load event
@event.listens_for(User, 'load')
def receive_load(target, context):
    print(f"User {target.id} loaded")

# Attribute events
@event.listens_for(User.email, 'set')
def receive_set(target, value, oldvalue, initiator):
    return value.lower()

# Session events
@event.listens_for(Session, 'before_commit')
def before_commit(session):
    print("About to commit")

@event.listens_for(Session, 'after_commit')
def after_commit(session):
    print("Committed")
```

### Validation with Decorators

```python
from sqlalchemy.orm import validates

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str]
    age: Mapped[int]
    
    @validates('email')
    def validate_email(self, key, email):
        if '@' not in email:
            raise ValueError("Invalid email address")
        return email.lower()
    
    @validates('age')
    def validate_age(self, key, age):
        if age < 0 or age > 150:
            raise ValueError("Invalid age")
        return age
```

---

## 13. Performance & Optimization

### N+1 Query Problem

```python
# BAD: N+1 queries
users = session.execute(select(User)).scalars().all()
for user in users:
    print(user.addresses)  # Query per user!

# GOOD: Eager loading
stmt = select(User).options(selectinload(User.addresses))
users = session.execute(stmt).scalars().all()
for user in users:
    print(user.addresses)  # No additional queries
```

### Query Optimization Strategies

```python
# 1. Select only needed columns
stmt = select(User.id, User.name).where(User.age > 25)

# 2. Use load_only for ORM
from sqlalchemy.orm import load_only
stmt = select(User).options(load_only(User.id, User.name))

# 3. Defer loading of large columns
from sqlalchemy.orm import defer
stmt = select(User).options(defer(User.large_blob_column))

# 4. Use exists() instead of count()
exists_stmt = select(User).where(User.name == 'Alice').exists()
has_alice = session.execute(select(exists_stmt)).scalar()

# 5. Batch operations
session.bulk_insert_mappings(User, [
    {'name': 'Alice', 'email': 'alice@example.com'},
    {'name': 'Bob', 'email': 'bob@example.com'}
])

session.bulk_update_mappings(User, [
    {'id': 1, 'name': 'Alice Updated'},
    {'id': 2, 'name': 'Bob Updated'}
])
```

### Bulk Operations

```python
# Bulk insert (bypasses ORM events)
session.bulk_insert_mappings(User, [
    {'name': 'User1', 'email': 'user1@example.com'},
    {'name': 'User2', 'email': 'user2@example.com'}
])

# Bulk update
session.bulk_update_mappings(User, [
    {'id': 1, 'name': 'Updated1'},
    {'id': 2, 'name': 'Updated2'}
])

# Bulk save objects
users = [User(name=f'User{i}') for i in range(1000)]
session.bulk_save_objects(users)

# Core bulk insert (fastest)
stmt = insert(users_table)
conn.execute(stmt, [
    {'name': 'User1', 'email': 'user1@example.com'},
    {'name': 'User2', 'email': 'user2@example.com'}
])
```

### Indexing

```python
from sqlalchemy import Index

class User(Base):
    __tablename__ = 'users'
    
    id: Mapped[int] = mapped_column(primary_key=True)
    email: Mapped[str] = mapped_column(unique=True, index=True)
    name: Mapped[str]
    age: Mapped[int]
    
    # Composite index
    __table_args__ = (
        Index('idx_name_age', 'name', 'age'),
        Index('idx_email_lower', func.lower(email)),  # Functional index
    )
```

### Query Result Caching

```python
from sqlalchemy import event
from sqlalchemy.orm import Query

# Simple caching decorator
cache = {}

def cache_query(fn):
    def wrapper(*args, **kwargs):
        key = str(args) + str(kwargs)
        if key not in cache:
            cache[key] = fn(*args, **kwargs)
        return cache[key]
    return wrapper

# Dogpile.cache integration
from dogpile.cache import make_region

region = make_region().configure(
    'dogpile.cache.memory',
    expiration_time=3600
)

@region.cache_on_arguments()
def get_user_by_email(email):
    return session.execute(
        select(User).where(User.email == email)
    ).scalar_one_or_none()
```

### Connection Pooling

```python
from sqlalchemy.pool import QueuePool, NullPool, StaticPool

# Custom pool configuration
engine = create_engine(
    'postgresql://user:pass@localhost/db',
    poolclass=QueuePool,
    pool_size=20,              # Number of connections to maintain
    max_overflow=0,            # Don't allow overflow
    pool_timeout=30,           # Timeout for getting connection
    pool_recycle=3600,         # Recycle connections after 1 hour
    pool_pre_ping=True,        # Test connection before using
    echo_pool=True             # Log pool operations
)

# No pooling (creates new connection each time)
engine = create_engine('postgresql://...', poolclass=NullPool)

# Static pool (single connection, for SQLite)
engine = create_engine('sqlite:///:memory:', poolclass=StaticPool)
```

### Profiling and Debugging

```python
# Enable SQL echo
engine = create_engine('postgresql://...', echo=True)

# Time queries
import time
from sqlalchemy import event

@event.listens_for(engine, "before_cursor_execute")
def before_cursor_execute(conn, cursor, statement, parameters, context, executemany):
    conn.info.setdefault('query_start_time', []).append(time.time())

@event.listens_for(engine, "after_cursor_execute")
def after_cursor_execute(conn, cursor, statement, parameters, context, executemany):
    total = time.time() - conn.info['query_start_time'].pop(-1)
    print(f"Query took {total:.4f}s: {statement[:100]}")

# Query analysis
stmt = select(User).where(User.age > 25)
print(stmt)  # Print SQL
print(stmt.compile())  # Compile with dialect
print(stmt.compile().params)  # Show parameters

# EXPLAIN query (PostgreSQL)
from sqlalchemy import text
result = session.execute(
    text("EXPLAIN ANALYZE SELECT * FROM users WHERE age > :age"),
    {"age": 25}
)
print(result.all())
```

### Batch Fetching

```python
# Fetch in batches
batch_size = 1000
offset = 0

while True:
    stmt = select(User).limit(batch_size).offset(offset)
    users = session.execute(stmt).scalars().all()
    
    if not users:
        break
    
    # Process batch
    for user in users:
        process_user(user)
    
    offset += batch_size
    session.expunge_all()  # Clear session to free memory
```

---

## 14. Migration with Alembic

### Installation and Setup

```bash
# Install Alembic
pip install alembic

# Initialize Alembic
alembic init alembic

# This creates:
# alembic/
#   versions/
#   env.py
#   script.py.mako
# alembic.ini
```

### Configuration

```python
# alembic/env.py
from sqlalchemy import engine_from_config, pool
from logging.config import fileConfig
from alembic import context
from myapp.models import Base  # Import your Base

# Alembic Config object
config = context.config

# Set up logging
fileConfig(config.config_file_name)

# Set target metadata
target_metadata = Base.metadata

def run_migrations_online():
    connectable = engine_from_config(
        config.get_section(config.config_ini_section),
        prefix='sqlalchemy.',
        poolclass=pool.NullPool,
    )

    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )

        with context.begin_transaction():
            context.run_migrations()
```

```ini
# alembic.ini
[alembic]
script_location = alembic
sqlalchemy.url = postgresql://user:password@localhost/dbname
```

### Creating Migrations

```bash
# Auto-generate migration from models
alembic revision --autogenerate -m "Add users table"

# Create empty migration
alembic revision -m "Manual changes"

# View current revision
alembic current

# View migration history
alembic history

# View specific migration
alembic show <revision>
```

### Running Migrations

```bash
# Upgrade to latest
alembic upgrade head

# Upgrade to specific revision
alembic upgrade <revision>

# Upgrade by relative number
alembic upgrade +1
alembic upgrade +2

# Downgrade
alembic downgrade -1
alembic downgrade <revision>
alembic downgrade base  # Downgrade all

# Show SQL without running
alembic upgrade head --sql
```

### Migration Script Structure

```python
"""Add users table

Revision ID: abc123
Revises: 
Create Date: 2024-01-01 12:00:00
"""
from alembic import op
import sqlalchemy as sa

# revision identifiers
revision = 'abc123'
down_revision = None
branch_labels = None
depends_on = None

def upgrade():
    # Create table
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), primary_key=True),
        sa.Column('name', sa.String(50), nullable=False),
        sa.Column('email', sa.String(100), unique=True),
        sa.Column('created_at', sa.DateTime(), server_default=sa.func.now())
    )
    
    # Create index
    op.create_index('idx_email', 'users', ['email'])
    
    # Add column to existing table
    op.add_column('users', sa.Column('age', sa.Integer()))
    
    # Alter column
    op.alter_column('users', 'name', new_column_name='full_name')
    
    # Create foreign key
    op.create_foreign_key(
        'fk_user_address',
        'addresses', 'users',
        ['user_id'], ['id']
    )

def downgrade():
    # Drop foreign key
    op.drop_constraint('fk_user_address', 'addresses', type_='foreignkey')
    
    # Drop index
    op.drop_index('idx_email', 'users')
    
    # Drop column
    op.drop_column('users', 'age')
    
    # Drop table
    op.drop_table('users')
```

### Common Migration Operations

```python
# Create table
op.create_table(
    'products',
    sa.Column('id', sa.Integer(), primary_key=True),
    sa.Column('name', sa.String(100)),
    sa.Column('price', sa.Numeric(10, 2))
)

# Drop table
op.drop_table('products')

# Add column
op.add_column('users', sa.Column('phone', sa.String(20)))

# Drop column
op.drop_column('users', 'phone')

# Rename column
op.alter_column('users', 'name', new_column_name='full_name')

# Change column type
op.alter_column('users', 'age', type_=sa.String())

# Add constraint
op.create_unique_constraint('uq_email', 'users', ['email'])
op.create_check_constraint('ck_age', 'users', 'age >= 18')

# Drop constraint
op.drop_constraint('uq_email', 'users', type_='unique')

# Create index
op.create_index('idx_name', 'users', ['name'])

# Drop index
op.drop_index('idx_name', 'users')

# Rename table
op.rename_table('old_name', 'new_name')

# Execute raw SQL
op.execute('UPDATE users SET status = "active"')

# Batch operations (for SQLite)
with op.batch_alter_table('users') as batch_op:
    batch_op.add_column(sa.Column('phone', sa.String(20)))
    batch_op.drop_column('old_field')
```

### Data Migrations

```python
from alembic import op
import sqlalchemy as sa
from sqlalchemy.sql import table, column

def upgrade():
    # Define table structure for data manipulation
    users = table('users',
        column('id', sa.Integer),
        column('status', sa.String)
    )
    
    # Update data
    op.execute(
        users.update().where(
            users.c.status == 'pending'
        ).values(status='active')
    )
    
    # Insert data
    op.bulk_insert(users, [
        {'id': 1, 'status': 'active'},
        {'id': 2, 'status': 'inactive'}
    ])
    
    # Delete data
    op.execute(
        users.delete().where(users.c.status == 'inactive')
    )
```

---

## Quick Reference: Common Patterns

### CRUD Operations

```python
# CREATE
user = User(name='Alice', email='alice@example.com')
session.add(user)
session.commit()

# READ
user = session.get(User, 1)
users = session.execute(select(User)).scalars().all()
user = session.execute(
    select(User).where(User.email == 'alice@example.com')
).scalar_one_or_none()

# UPDATE
user = session.get(User, 1)
user.name = 'Alice Updated'
session.commit()

# DELETE
user = session.get(User, 1)
session.delete(user)
session.commit()
```

### Common Query Patterns

```python
# Filter with multiple conditions
stmt = select(User).where(
    User.age > 25,
    User.email.like('%@example.com'),
    User.is_active == True
)

# Order and limit
stmt = select(User).order_by(User.name).limit(10)

# Pagination
page = 1
page_size = 10
stmt = select(User).limit(page_size).offset((page - 1) * page_size)

# Count
count = session.execute(select(func.count(User.id))).scalar()

# Exists
exists = session.execute(
    select(User).where(User.email == 'alice@example.com').exists()
).scalar()

# Join and filter
stmt = select(User).join(Address).where(
    Address.email.like('%@example.com')
)

# Eager load relationships
stmt = select(User).options(selectinload(User.addresses))
```

### Session Patterns

```python
# Context manager (recommended)
with Session(engine) as session:
    user = User(name='Alice')
    session.add(user)
    session.commit()

# Try-except-finally
session = Session(engine)
try:
    user = User(name='Alice')
    session.add(user)
    session.commit()
except Exception:
    session.rollback()
    raise
finally:
    session.close()

# Scoped session (thread-local)
from sqlalchemy.orm import scoped_session

SessionLocal = scoped_session(sessionmaker(bind=engine))
session = SessionLocal()
# ... use session ...
SessionLocal.remove()  # Clean up
```

---

## Best Practices

1. **Always use context managers** for sessions and connections
2. **Use eager loading** to avoid N+1 query problems
3. **Enable pool_pre_ping** for long-lived applications
4. **Use transactions explicitly** for data consistency
5. **Create indexes** on frequently queried columns
6. **Use bulk operations** for large datasets
7. **Defer large columns** that aren't always needed
8. **Use select() syntax** (SQLAlchemy 2.0 style) instead of Query
9. **Never commit in library code** - let the caller control transactions
10. **Use Alembic** for all schema changes in production

---

## Common Gotchas

1. **Detached instances**: Objects become detached after session.close()
2. **Lazy loading errors**: Accessing relationships after session closed
3. **Identity map**: Session keeps objects in memory - use expunge_all() for large batches
4. **Autoflush**: Queries trigger flush - can cause unexpected INSERTs/UPDATEs
5. **expire_on_commit**: Objects expire after commit - access triggers SELECT
6. **Cascade delete**: Relationships cascade by default - configure carefully
7. **Bind parameters**: Always use parameters, never string formatting
8. **Connection leaks**: Always close sessions/connections
9. **Transaction isolation**: Know your database's default isolation level
10. **Migration conflicts**: Always pull latest migrations before creating new ones

---

This reference covers the core concepts and patterns in SQLAlchemy. For specific advanced topics or edge cases, consult the official documentation at https://docs.sqlalchemy.org/
