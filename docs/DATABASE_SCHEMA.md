# Biteme Database Schema

## Overview
PostgreSQL database schema for Biteme Food Delivery & Quick Commerce platform.

## Core Tables

### users
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  phone VARCHAR(20) UNIQUE,
  avatar_url VARCHAR(500),
  role ENUM('customer', 'restaurant', 'delivery', 'admin'),
  status ENUM('active', 'inactive', 'suspended'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### restaurants
```sql
CREATE TABLE restaurants (
  id UUID PRIMARY KEY,
  user_id UUID FOREIGN KEY REFERENCES users(id),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  cuisine_types JSON,
  rating DECIMAL(3, 2),
  min_order_amount DECIMAL(10, 2),
  delivery_fee DECIMAL(10, 2),
  is_verified BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### products
```sql
CREATE TABLE products (
  id UUID PRIMARY KEY,
  restaurant_id UUID FOREIGN KEY REFERENCES restaurants(id),
  name VARCHAR(255) NOT NULL,
  price DECIMAL(10, 2) NOT NULL,
  discount_price DECIMAL(10, 2),
  category VARCHAR(100),
  availability BOOLEAN DEFAULT TRUE,
  rating DECIMAL(3, 2),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### orders
```sql
CREATE TABLE orders (
  id UUID PRIMARY KEY,
  user_id UUID FOREIGN KEY REFERENCES users(id),
  restaurant_id UUID FOREIGN KEY REFERENCES restaurants(id),
  delivery_partner_id UUID FOREIGN KEY REFERENCES users(id),
  total_amount DECIMAL(10, 2) NOT NULL,
  status ENUM('pending', 'confirmed', 'preparing', 'ready', 'out_for_delivery', 'delivered', 'cancelled'),
  payment_status ENUM('pending', 'completed', 'failed', 'refunded'),
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### order_items
```sql
CREATE TABLE order_items (
  id UUID PRIMARY KEY,
  order_id UUID FOREIGN KEY REFERENCES orders(id),
  product_id UUID FOREIGN KEY REFERENCES products(id),
  quantity INT NOT NULL,
  unit_price DECIMAL(10, 2) NOT NULL,
  total_price DECIMAL(10, 2) NOT NULL
);
```

### addresses
```sql
CREATE TABLE addresses (
  id UUID PRIMARY KEY,
  user_id UUID FOREIGN KEY REFERENCES users(id),
  type ENUM('home', 'work', 'other'),
  address_line_1 VARCHAR(255) NOT NULL,
  city VARCHAR(100) NOT NULL,
  latitude DECIMAL(10, 8),
  longitude DECIMAL(11, 8),
  is_default BOOLEAN DEFAULT FALSE
);
```

### payments
```sql
CREATE TABLE payments (
  id UUID PRIMARY KEY,
  order_id UUID FOREIGN KEY REFERENCES orders(id),
  amount DECIMAL(10, 2) NOT NULL,
  method ENUM('card', 'wallet', 'upi', 'cash'),
  status ENUM('pending', 'completed', 'failed', 'refunded'),
  transaction_id VARCHAR(255)
);
```

### delivery_partners
```sql
CREATE TABLE delivery_partners (
  id UUID PRIMARY KEY,
  user_id UUID FOREIGN KEY REFERENCES users(id),
  vehicle_type ENUM('bike', 'scooter', 'car'),
  rating DECIMAL(3, 2),
  total_deliveries INT DEFAULT 0,
  is_verified BOOLEAN DEFAULT FALSE
);
```

### promos
```sql
CREATE TABLE promos (
  id UUID PRIMARY KEY,
  code VARCHAR(50) UNIQUE NOT NULL,
  discount_type ENUM('percentage', 'fixed'),
  discount_value DECIMAL(10, 2),
  min_order_amount DECIMAL(10, 2),
  valid_from TIMESTAMP,
  valid_until TIMESTAMP,
  is_active BOOLEAN DEFAULT TRUE
);
```

### reviews
```sql
CREATE TABLE reviews (
  id UUID PRIMARY KEY,
  user_id UUID FOREIGN KEY REFERENCES users(id),
  restaurant_id UUID FOREIGN KEY REFERENCES restaurants(id),
  rating INT NOT NULL,
  comment TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### notifications
```sql
CREATE TABLE notifications (
  id UUID PRIMARY KEY,
  user_id UUID FOREIGN KEY REFERENCES users(id),
  title VARCHAR(255) NOT NULL,
  message TEXT NOT NULL,
  type ENUM('order', 'promo', 'payment', 'system'),
  is_read BOOLEAN DEFAULT FALSE
);
```

## Key Relationships

- Users → Addresses (1:N)
- Users → Orders (1:N)
- Users → Reviews (1:N)
- Restaurants → Products (1:N)
- Restaurants → Orders (1:N)
- Orders → Order Items (1:N)
- Orders → Payments (1:N)
- Delivery Partners → Orders (1:N)

## Indexes

```sql
CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_phone ON users(phone);
CREATE INDEX idx_restaurants_user_id ON restaurants(user_id);
CREATE INDEX idx_products_restaurant_id ON products(restaurant_id);
CREATE INDEX idx_orders_user_id ON orders(user_id);
CREATE INDEX idx_orders_status ON orders(status);
CREATE INDEX idx_addresses_user_id ON addresses(user_id);
```
