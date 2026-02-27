Initialize or verify the Rust backend project for DIU OS.

If backend/ directory doesn't exist, create it with:
- Rust + Axum framework
- DDD (Domain-Driven Design) structure:
  ```
  backend/
  ├── Cargo.toml
  ├── src/
  │   ├── main.rs
  │   ├── config.rs
  │   ├── domain/          # Business logic (entities, value objects)
  │   ├── application/     # Use cases, services
  │   ├── infrastructure/  # DB, external APIs, blockchain
  │   │   ├── database/    # PostgreSQL + Supabase
  │   │   ├── blockchain/  # Arbitrum contract interaction
  │   │   └── ai/          # MCP protocol servers
  │   ├── api/             # REST + WebSocket routes (Axum)
  │   │   ├── routes/
  │   │   ├── middleware/
  │   │   └── handlers/
  │   └── errors.rs
  ├── migrations/          # SQL migrations
  └── tests/
  ```
- Dependencies: axum, tokio, serde, sqlx, tower-http
- Basic health check endpoint at GET /health
- CORS and logging middleware

If backend/ exists, verify structure and suggest improvements.
