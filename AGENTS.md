# AGENTS.md

This file provides guidance to agents when working with code in this repository.

## Project Overview
Dual-protocol booking system: FastAPI backend serves both REST API and MCP (Model Context Protocol) from single server on port 8080.

## Critical Non-Obvious Patterns

### Backend Architecture
- **MCP Server Creation Order**: MCP server MUST be created before FastAPI app to properly combine lifespans (see server.py:16)
- **Service Layer Returns**: Business logic functions return `Union[SuccessType, ErrorResponse]` - always check `isinstance(result, ErrorResponse)` before using
- **Session Management Split**: MCP tools manually manage sessions with `SessionLocal()` + try/finally; REST endpoints use FastAPI dependency injection
- **Booking Validation**: `book_flight()` requires BOTH `user_id` AND `name` parameters to match database - not just user_id alone

### Frontend Specifics
- **User Persistence**: User state stored in localStorage with key `'galaxium_user'` (not session storage)
- **Environment Variables**: Uses Vite's `import.meta.env.VITE_API_URL`, not `process.env`
- **Type Convention**: All types use snake_case to exactly match Python backend (e.g., `flight_id`, `user_id`)
- **Error Structure**: Backend returns `ErrorResponse` with `error_code` field - handle this specifically

### Testing
- **Test Database**: Uses in-memory SQLite with `StaticPool` - tests monkey-patch `SessionLocal` globally
- **Test Isolation**: Each test gets fresh database via `db_session` fixture with automatic cleanup
- **Run Tests**: From `booking_system_backend/` directory, just run `pytest` (no need for `python -m pytest`)

## Commands

### Backend (from booking_system_backend/)
```bash
python server.py              # Start server (port 8080)
pytest                        # Run all tests
pytest tests/test_services.py # Run specific test file
pytest -v                     # Verbose output
```

### Frontend (from booking_system_frontend/)
```bash
npm run dev                   # Start dev server (port 5173)
npm run build                 # Production build
npm run lint                  # Run ESLint
```

### Root Directory
```bash
./start.sh                    # Start both backend and frontend (Unix/Mac)
```

## Code Style

### Backend
- Use type hints: `def func() -> ReturnType | ErrorResponse:`
- Service functions return union types with ErrorResponse
- Manual session management in MCP tools, dependency injection in REST

### Frontend
- Functional components with hooks only
- snake_case for all API-related types (matches backend)
- Use `import.meta.env` for environment variables (Vite-specific)