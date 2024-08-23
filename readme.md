
# Lecture: Introduction to Object-Relational Mappers (ORMs) with Flask and SQLAlchemy

## Useful Links:

[SQLAlchemy Quickstart Guide](https://docs.sqlalchemy.org/en/20/orm/quickstart.html)

[SQLAlchemy Mapped Class Configuration](https://docs.sqlalchemy.org/en/20/orm/mapper_config.html)

[SQLAlchemy Relationship Configuration](https://docs.sqlalchemy.org/en/20/orm/relationships.html)

[SQLAlchemy Querying Guide](https://docs.sqlalchemy.org/en/20/orm/queryguide/index.html)

[Understanding Connection Strings](https://www.prisma.io/dataguide/postgresql/short-guides/connection-uris)

## Object Relational Mappers (ORM)
**Definition**: ORMs serve as a link between our application/API and its database, managing the interactions between the two.

### Benefits of ORMs
- **Ease of Use**: No longer needing to write complex SQL queries.
- **Efficiency**: ORMs optimize database interactions.
- **Maintainability**: Working in tandem with database schemas, an ORM can seamlessly handle changes to fields of the database tables.

## Flask-SQLAlchemy
**Definition**: Flask-SQLAlchemy is an extension of Flask that serves as an ORM, allowing us to connect from our Flask app to our database, streamlining these interactions.

### Integrating Flask-SQLAlchemy
1. **Create a virtual environment in your project folder, and activate the venv**
    ```sh
    python -m venv venv
    source venv/bin/activate  # On Windows use `venv\Scripts\activate`
    ```
2. **Install all necessary packages**
    ```sh
    pip install flask flask-sqlalchemy flask-marshmallow marshmallow-sqlalchemy mysql-connector-python
    ```

### Getting Started
1. **Import necessary modules**
    ```python
    from flask import Flask, jsonify, request
    from flask_sqlalchemy import SQLAlchemy
    from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
    from flask_marshmallow import Marshmallow
    import datetime
    from typing import List
    from marshmallow import ValidationError, fields
    from sqlalchemy import select, delete
    ```

2. **Instantiate the Flask app**
    ```python
    app = Flask(__name__)
    ```

3. **Configure the Database URI for your app**
    ```python
    app.config['SQLALCHEMY_DATABASE_URI'] = 'mysql+mysqlconnector://<user>:<password>@<host>/<database_name>'
    ```
4. **Instantiate SQLAlchemy using the app**
    ```python
    db = SQLAlchemy(app)
    ma = Marshmallow(app)
    ```

5. **Create a Base class that inherits from `DeclarativeBase`**

    When declaring models (tables), we define module-level constructs that will form the structures which we will be querying from the database.

    This structure, known as `Declarative Mapping`, starts by defining a base class called `Base`, inheriting the `DeclarativeBase` class

    ```python
    class Base(DeclarativeBase):
        pass
    ```
    Individual mapped classes are then created by making subclasses of Base. A mapped class typically refers to a single particular database table, the name of which is indicated by using the `__tablename__` class-level attribute.


### Create Classes to Reflect DB Tables
- Each class represents a table in the database.
- Map each attribute to its associated column in the table.

Example:
```python
class User(db.Model):
    __tablename__ = 'users'
    id = mapped_column(db.Integer, primary_key=True)
    username = mapped_column(db.String(150), unique=True, nullable=False)
    email = mapped_column(db.String(150), unique=True, nullable=False)
    created_at = mapped_column(db.DateTime, default=datetime.datetime.utcnow)
```

### Creating Relationships
- **One-to-One Relationship**: Map to a single object in the related table and vice-versa.
    ```python
    class UserProfile(db.Model):
        __tablename__ = 'user_profiles'
        id = mapped_column(db.Integer, primary_key=True)
        user_id = mapped_column(db.Integer, db.ForeignKey('users.id'), unique=True)
        user = db.relationship('User', backref=db.backref('profile', uselist=False))
    ```

- **One-to-Many Relationship**: Map to a list of objects in the related table, but the related table will map back to a single object.
    ```python
    class Post(db.Model):
        __tablename__ = 'posts'
        id = mapped_column(db.Integer, primary_key=True)
        user_id = mapped_column(db.Integer, db.ForeignKey('users.id'))
        user = db.relationship('User', backref=db.backref('posts', lazy=True))
    ```

### Many-to-Many Relationships
1. **Define an association table**
    ```python
    orders_products = db.Table('orders_products',
        mapped_column('order_id', db.Integer, db.ForeignKey('orders.id'), primary_key=True),
        mapped_column('product_id', db.Integer, db.ForeignKey('products.id'), primary_key=True)
    )
    ```

2. **Create the `Order` and `Product` classes**
    ```python
    class Order(db.Model):
        __tablename__ = 'orders'
        id = mapped_column(db.Integer, primary_key=True)
        products = db.relationship('Product', secondary=orders_products, backref=db.backref('orders', lazy=True))

    class Product(db.Model):
        __tablename__ = 'products'
        id = mapped_column(db.Integer, primary_key=True)
        name = mapped_column(db.String(100), nullable=False)
    ```

## Conclusion
ORMs, like Flask-SQLAlchemy, provide a powerful toolset for interacting with databases in a more Pythonic and efficient manner. They abstract away the complexity of direct SQL queries, allowing developers to work with database records as if they were regular Python objects.

## Resources
- [Flask-SQLAlchemy Documentation](https://flask-sqlalchemy.palletsprojects.com/)
- [SQLAlchemy Documentation](https://www.sqlalchemy.org/)
- [Marshmallow Documentation](https://marshmallow.readthedocs.io/)
- [GitHub Repository](https://github.com/dkatina/REST-API-146)
