# PLANNING.md

## 1. Project Overview

*   **Project Name**: MCP Lawyer / Pathways Law Practice Management Server (Primary name defined in `app/config.py`)
*   **Objective**: A Python-based MCP (Model Context Protocol) server designed for legal applications, providing AI-powered assistance and practice management tools.
*   **Core Technology**: FastAPI backend served with Uvicorn.

## 2. Architecture

*   **Framework**: FastAPI
*   **Server**: Uvicorn, launched via `server.py`.
*   **Modularity**:
    *   API endpoints are defined in modules within `app/routes/`.
    *   Business logic is encapsulated in service classes within `app/services/`.
    *   Data validation and serialization are handled by Pydantic models, likely located in `app/models/`.
*   **Configuration**:
    *   Environment variables are loaded from a `.env` file.
    *   Application settings (including detailed OpenAI model selection, TTLs, server info) are defined in `app/config.py` using Pydantic's `BaseSettings`. An instance named `settings` is created here and used by `app/main.py` and various services. `app/config.py` is the single source of truth for configuration.
*   **Service Management**:
    *   Services are initialized within the `lifespan` context manager in `app/main.py`.
    *   Initialized service instances (e.g., `AIProcessor`, `MemoryService`) are stored in `app.state` and accessed by route handlers via `request.app.state.{service_name}` using FastAPI's dependency injection.
*   **Data Storage**:
    *   Currently, some services like `MemoryService` use in-memory Python dictionaries for data storage (e.g., `self.memories`). Code comments indicate this is for development and would be replaced by a persistent database in production.
    *   `app/config.py` includes optional settings for Redis and Supabase, suggesting these are potential candidates for persistent storage.
*   **Middleware**:
    *   CORS middleware is configured in `app/main.py` to allow cross-origin requests (e.g., from the frontend).
    *   Custom middleware for detailed request/response logging is also present in `app/main.py`.

## 3. Key Components

*   **`server.py`**: Main entry point; loads environment variables and starts the Uvicorn server for the FastAPI app.
*   **`app/main.py`**: Initializes the FastAPI application, sets up middleware (CORS, logging), manages service lifecycles using `asynccontextmanager lifespan`, and includes all routers from `app/routes/`.
*   **`app/config.py`**: Defines the `Settings` class (Pydantic `BaseSettings`) for application-wide configuration, including `OpenAIModel` enum, `ModelTaskConfig` for nuanced AI model selection, server details, and TTLs. This is the single source of truth for all configurations. An instance `settings` is created in this file and imported by `app/main.py`.
*   **`app/routes/`**: Contains modules for different API resources (e.g., `memory_routes.py`, `role_routes.py`, `client_intake_routes.py`). Each module typically defines an `APIRouter` with specific endpoints.
*   **`app/services/`**: Contains service classes that encapsulate business logic (e.g., `MemoryService.py`, `AIProcessor.py`, `ClauseLibraryService.py`). These services interact with data sources and perform core operations.
*   **`app/models/`**: (Assumed location based on imports like `app.models.memory`) Contains Pydantic models for request/response data validation and serialization.

## 4. Naming Conventions (Observed)

*   **Python Files**: `snake_case.py` (e.g., `memory_service.py`).
*   **Classes**: `PascalCase` (e.g., `MemoryService`, `AIProcessor`).
*   **Functions/Methods**: `snake_case` (e.g., `store_memory`, `generate_response`).
*   **Variables**: `snake_case`.
*   **API Endpoints**: Generally follow RESTful principles. Prefixed with `/api/v1` (defined in `app/config.py`). Resource names are plural (e.g., `/memories`, `/roles`).

## 5. Extension Methodology

*   **New Features**: Should generally follow the established pattern:
    1.  Define Pydantic models for request/response data in `app/models/`.
    2.  Create a new service class in `app/services/` to encapsulate the business logic.
    3.  Initialize the new service in `app/main.py`'s `lifespan` manager and add it to `app.state`.
    4.  Create a new route module in `app/routes/` with an `APIRouter`. Define endpoints that use the new service (accessed via dependency injection).
    5.  Include the new router in `app/main.py`.
*   **Domain-Specific Logic**: Currently, legal domain-specific services (e.g., `ClauseLibraryService`, `ContractAnalysisService`) are directly within `app/services/`. For future large-scale extensions into new domains (e.g., education, finance), consider if a more modular structure like `/src/extensions/{domain}` (as per global rules, though originally for TypeScript) would be beneficial, or if sub-packages within `app/services/` and `app/routes/` are sufficient.

## 6. Points to Address / Architectural Considerations

*   **Configuration Consolidation**: (Completed) `app/config.py` has been established as the single source of truth for application settings. The redundant `app/settings.py` file has been removed, and relevant services (`AIProcessor`, `ClientIntakeService`, `ContractAnalysisService`) were updated to rely solely on the settings instance from `app/config.py`.
*   **Persistent Storage**: Transition services using in-memory storage (like `MemoryService`) to a persistent database solution (e.g., Supabase, as mentioned in global rules, or Redis, for which settings exist).
*   **Error Handling**: Standardize error handling in services. For example, `AIProcessor.generate_response()` currently returns a formatted string on error; it should ideally raise specific HTTPExceptions or custom exceptions that route handlers can convert into appropriate API responses.
*   **Logging**: The initial pass of replacing `print()` statements with `logger` calls (configured in `app/main.py`) across services and routes is complete. The next step is to write unit tests to verify the logging implementation, ensuring consistent usage of appropriate log levels (e.g., `logger.debug()`, `logger.info()`, `logger.warning()`, `logger.error()`, `logger.exception()`).
*   **Service Abstraction for OpenAI**: Consider if `AIProcessor` should be the sole direct interface to OpenAI, or if `OpenAIService` (which also exists) has a distinct role. Clarify their responsibilities.
*   **Large Service Files**: Some service files are very large (e.g., `contract_analysis_service.py`, `client_intake_service.py`). While not an immediate issue, monitor for maintainability and consider refactoring into smaller, more focused modules if they become unwieldy.
*   **Testing Strategy**: Define and implement a testing strategy, including unit tests for services (mocking external dependencies like OpenAI) and integration tests for API endpoints.

This `PLANNING.md` should serve as a good starting point for guiding development and addressing key architectural aspects of the MCP Lawyer server.
