# ChainTrace - Blockchain-Based Supply Chain Tracking

<div align="center">
  <strong>A full-stack blockchain supply chain transparency solution with real-time tracking, immutable records, and QR code verification</strong>
</div>

## Overview

ChainTrace is a comprehensive supply chain management system that leverages blockchain technology to provide transparency, authenticity verification, and tamper-proof product tracking. It combines a modern React frontend with a Flask/Python backend and PostgreSQL database for robust supply chain management.

### Key Features

- **Blockchain Integration**: Immutable transaction records and product authentication
- **Real-time Tracking**: Monitor products through every stage of the supply chain
- **QR Code Scanning**: Verify product authenticity via QR codes
- **Dashboard Analytics**: Real-time statistics and product overview
- **Multi-stage Tracking**: Track products through manufacturing, transit, storage, and delivery
- **Activity Logs**: Complete blockchain-verified audit trails
- **PostgreSQL Database**: Persistent storage with efficient querying

## Architecture

```
┌─────────┬─────────┬─────────┐
│   React UI  │ Flask API  │ PostgreSQL │
│ (Frontend)  │ (Backend)  │ (Database) │
├─────────┼─────────┼─────────┤
│                                      │
│         Blockchain Network           │
│    (Web3/Ethereum Compatible)       │
└─────────────────────────────┘
```

## Project Structure

```
chaintrace-app/
├── backend/
│   ├── config/
│   │   └── database.py          # Database configuration
│   ├── models/
│   │   ├── product.py          # Product & Activity models
│   │   └── __init__.py
│   ├── routes/
│   │   ├── products.py         # Product registration endpoints
│   │   ├── tracking.py         # Tracking endpoints
│   │   ├── verification.py     # Verification endpoints
│   │   └── __init__.py
│   ├── services/
│   │   ├── blockchain_service.py  # Blockchain operations
│   │   ├── product_service.py     # Business logic
│   │   └── __init__.py
│   ├── app.py                  # Flask application
│   ├── requirements.txt        # Dependencies
│   └── .env                    # Environment variables
├── frontend/
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   ├── hooks/
│   │   └── App.jsx
│   └── package.json
├── docker-compose.yml      # Docker orchestration
└── README.md
```

## Tech Stack

### Backend
- **Framework**: Flask 2.3.3
- **Database**: PostgreSQL with SQLAlchemy ORM
- **Blockchain**: Web3.py (Ethereum integration)
- **API**: RESTful API with Flask-CORS
- **Validation**: Pydantic for data validation

### Frontend
- **Framework**: React with Vite
- **Styling**: Tailwind CSS
- **State Management**: React Hooks
- **QR Code**: qrcode.react library

### Database Schema

#### Products Table
```sql
CREATE TABLE products (
    id VARCHAR PRIMARY KEY,
    name VARCHAR NOT NULL,
    batch_number VARCHAR NOT NULL,
    manufacturing_date DATETIME NOT NULL,
    origin VARCHAR NOT NULL,
    category VARCHAR NOT NULL,
    description TEXT,
    owner VARCHAR NOT NULL,
    status VARCHAR DEFAULT 'manufactured',
    blockchain_hash VARCHAR UNIQUE NOT NULL,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    updated_at DATETIME DEFAULT CURRENT_TIMESTAMP
);
```

#### Activities Table
```sql
CREATE TABLE activities (
    id SERIAL PRIMARY KEY,
    product_id VARCHAR NOT NULL,
    event_type VARCHAR NOT NULL,
    location VARCHAR NOT NULL,
    handler VARCHAR NOT NULL,
    timestamp DATETIME DEFAULT CURRENT_TIMESTAMP,
    details TEXT,
    blockchain_tx_hash VARCHAR UNIQUE,
    created_at DATETIME DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (product_id) REFERENCES products(id)
);
```

## Installation

### Prerequisites
- Python 3.8+
- Node.js 16+
- PostgreSQL 12+
- Git

### Backend Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/Thamizh-02/ChainTrace-Blockchain-SupplyChain.git
   cd ChainTrace-Blockchain-SupplyChain
   ```

2. **Create virtual environment**
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\\Scripts\\activate
   ```

3. **Install dependencies**
   ```bash
   cd backend
   pip install -r requirements.txt
   ```

4. **Configure environment variables**
   ```bash
   cp .env.example .env
   # Edit .env with your database credentials
   ```

5. **Initialize database**
   ```bash
   python -c "from config.database import Base, engine; Base.metadata.create_all(bind=engine)"
   ```

6. **Run backend**
   ```bash
   python app.py
   ```

### Frontend Setup

1. **Navigate to frontend**
   ```bash
   cd ../frontend
   ```

2. **Install dependencies**
   ```bash
   npm install
   ```

3. **Configure API endpoint**
   ```bash
   # Create .env file
   VITE_API_URL=http://localhost:5000
   ```

4. **Run development server**
   ```bash
   npm run dev
   ```

## API Endpoints

### Product Management
- `POST /api/products/register` - Register new product
- `GET /api/products/<id>` - Get product details
- `GET /api/products` - List all products

### Tracking
- `GET /api/tracking/track/<product_id>` - Get product history
- `POST /api/tracking/update-status` - Update product status

### Verification
- `GET /api/verification/verify/<product_id>` - Verify product authenticity

### Dashboard
- `GET /api/dashboard/stats` - Get dashboard statistics

## Docker Deployment

```bash
# Build and run with Docker Compose
docker-compose up -d
```

## Database Queries

### Get Products by Status
```sql
SELECT * FROM products WHERE status = 'in-transit';
```

### Get Product History
```sql
SELECT * FROM activities WHERE product_id = 'PROD-001' ORDER BY timestamp DESC;
```

### Dashboard Statistics
```sql
SELECT 
  COUNT(*) as total_products,
  SUM(CASE WHEN status = 'in-transit' THEN 1 ELSE 0 END) as in_transit,
  SUM(CASE WHEN status = 'stored' THEN 1 ELSE 0 END) as in_storage,
  SUM(CASE WHEN status = 'delivered' THEN 1 ELSE 0 END) as delivered
FROM products;
```

## Usage Examples

### Register a Product
```bash
curl -X POST http://localhost:5000/api/products/register \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Organic Coffee Beans",
    "batch_number": "BCH-2024-001",
    "manufacturing_date": "2024-01-15T10:00:00",
    "origin": "Colombia",
    "category": "Agricultural",
    "description": "Premium organic coffee beans",
    "owner": "CoffeeMakers Ltd"
  }'
```

### Track a Product
```bash
curl http://localhost:5000/api/tracking/track/PROD-001
```

### Verify Product
```bash
curl http://localhost:5000/api/verification/verify/PROD-001
```

## Key Features Implementation

### Blockchain Hashing
Each product gets a unique immutable hash:
```python
blockchain_hash = Web3.keccak(text=product_data_string).hex()
```

### Activity Logging
Every status change is logged:
```python
activity = Activity(
    product_id=product_id,
    event_type=event_type,  # manufactured, shipped, stored, delivered
    location=location,
    handler=handler,
    blockchain_tx_hash=tx_hash
)
```

## Development

### Running Tests
```bash
pytest backend/tests/
```

### Database Migrations
```bash
# Using Alembic (if configured)
alembic upgrade head
```

## Performance Optimization

- Database indexes on frequently queried columns
- Connection pooling with SQLAlchemy
- CORS optimization for API requests
- Frontend code splitting with Vite
- Lazy loading of product images

## Security Considerations

- Environment variables for sensitive data
- SQL injection prevention via ORM
- CORS validation
- Input validation with Pydantic
- Blockchain-verified integrity

## Troubleshooting

### Database Connection Issues
```bash
# Check PostgreSQL is running
psql -U postgres -h localhost

# Verify DATABASE_URL in .env
echo $DATABASE_URL
```

### Port Already in Use
```bash
# Change port in Flask app
app.run(port=5001)
```

## Contributing

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit changes (`git commit -m 'Add amazing feature'`)
4. Push to branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

## Future Enhancements

- [ ] Smart contract integration
- [ ] Mobile app (iOS/Android)
- [ ] Advanced analytics dashboard
- [ ] IoT sensor integration
- [ ] Machine learning for anomaly detection
- [ ] Multi-chain support
- [ ] API rate limiting
- [ ] Real-time WebSocket updates

## License

MIT License - see LICENSE file for details

## Contact

**Developer**: Thamizh-02  
**GitHub**: [@Thamizh-02](https://github.com/Thamizh-02)  
**Email**: your-email@example.com

## Acknowledgments

- Built with Flask, React, and PostgreSQL
- Blockchain integration via Web3.py
- UI inspired by modern supply chain solutions
- Database design based on industry best practices

---

<div align="center">
  <strong>ChainTrace: Bringing Transparency to Supply Chains</strong>
</div>
