# TASK.md

This document tracks tasks for the MCP Lawyer server project.

## Discovered During Work (2025-05-30 - Initial Review by Cascade)
*   Dual configuration files (`app/config.py` and `app/settings.py`) with `AIProcessor` using the simpler `app/settings.py`.
*   `MemoryService` uses in-memory storage.
*   `AIProcessor` error handling returns formatted strings instead of raising exceptions.
*   `print()` statements used for debugging in services instead of the configured logger.
*   Several large service files.

## Current Tasks

### Priority #1: OpenAI Agents SDK Integration - Phase 1

*   `[DONE] 2025-05-30`: **Setup & Initial Configuration for Agents SDK**:
    *   Objective: Prepare the project environment for the OpenAI Agents SDK.
    *   Actions:
        1.  Add `openai-agents` to `requirements.txt`.
        2.  Run `pip install -r requirements.txt` (or ensure it's installed).
        3.  Review `app/config.py` for any immediate new settings required for basic agent definitions (e.g., default agent instructions, model choices for agents). Add placeholders if necessary.
*   `[DONE] 2025-05-30`: **Create `AgentOrchestrationService`**:
    *   Objective: Establish a dedicated service for managing and running AI agents built with the SDK.
    *   Actions:
        1.  Create `app/services/agent_orchestration_service.py`.
        2.  Define a basic structure for the `AgentOrchestrationService` class. This service will later house agent definitions and runner logic.
        3.  Initialize this service in `app/main.py`'s `lifespan` manager and add it to `app.state`.
*   `[DONE] 2025-05-30`: **Develop Initial Function Tools (3 examples: `get_clauses`, `search_case_law`, `format_case_citation`)**:
    *   Objective: Convert a few existing service methods into agent-usable tools.
    *   Actions:
        1.  Identify 2-3 relatively simple, high-value methods from existing services (e.g., `ClauseLibraryService.search_clauses`, `LegalResearchService.find_relevant_statutes`, `ClientIntakeService.get_form_questions`).
        2.  In their respective service files (or in `AgentOrchestrationService` as wrappers if preferred for modularity initially), apply the `@tool` decorator from `agents` SDK.
        3.  Ensure method parameters use Pydantic models (from `app/models/`) or clear type hints for automatic schema generation.
*   `[DONE] 2025-05-30`: **Implement Basic Agent Workflow in `AgentOrchestrationService`**:
    *   Objective: Create a first, simple agent that utilizes one or more of the new function tools.
    *   Actions:
        1.  Within `AgentOrchestrationService`, define a method (e.g., `run_legal_research_agent`).
        2.  Inside this method, instantiate an `Agent` with basic instructions (e.g., "You are a legal research assistant. Use available tools to answer questions.").
        3.  Make the newly created function tools available to this agent.
        4.  Use `Runner.run_sync` (or async equivalent) to execute the agent with a sample query that would require using a tool (e.g., "Find clauses related to 'indemnification'.").
*   `[DONE] 2025-05-30`: **Create API Endpoint for Basic Agent Workflow**:
    *   Objective: Expose the basic agent workflow via an API endpoint.
    *   Actions:
        1.  Create a new router file, e.g., `app/routes/agent_routes.py` (if it doesn't exist).
        2.  Define an `APIRouter` in this file.
        3.  Add a POST endpoint (e.g., `/api/v1/agents/perform-research`) that takes a user query.
        4.  This endpoint should call the corresponding method in `AgentOrchestrationService` (e.g., `run_legal_research_agent`) and return the agent's final output.
        5.  Include this new router in `app/main.py`.
*   `[TODO] 2025-05-30`: **Initial Testing & Explore Tracing**:
    *   Objective: Verify the end-to-end functionality of the basic agent workflow and familiarize with SDK tracing.
    *   Actions:
        1.  Manually test the new API endpoint with sample queries.
        2.  Verify that the agent correctly calls the intended tools and produces a relevant response.
        3.  Investigate how to enable and view the SDK's built-in tracing to understand the agent's execution flow.
*   `[TODO] 2025-05-30`: **Internal Documentation Update for Agents SDK Integration**:
    *   Objective: Document the initial agent integration approach for team reference.
    *   Actions:
        1.  Add a section to `PLANNING.md` or create a new internal note detailing:
            *   The role of `AgentOrchestrationService`.
            *   How to convert existing service methods into function tools.
            *   Basic principles for defining new agents.


### Configuration & Setup

### Core Functionality Enhancements

*   `[TODO] 2025-05-30`: **Review OpenAI Agents SDK for Potential Integration**:
    *   Objective: Evaluate the OpenAI Agents SDK to understand its capabilities and assess how it could be used to add more models or agentic features to the MCP Lawyer project.
    *   Action: Read SDK documentation, consider integration points with the existing FastAPI architecture, and report findings.
*   `[DONE] 2025-05-30`: **Standardize Logging**:
    *   Objective: Use the `logging` module consistently throughout the application.
    *   Action: Replaced `print()` statements with `logger` calls in services and routes. Leveraged the logger configured in `app/main.py`.
    *   **Actions Taken (2025-05-30)**:
        *   `app/services/memory_service.py`: Updated to use `logger.info()`.
        *   `app/services/predictive_analysis_service.py`: Implemented `logger` calls.
        *   `app/services/law_practice_service.py`: Replaced `print()` with `logger.info()`.
        *   `app/services/role_service.py`: Updated to use `logger` calls.
        *   `app/services/legal_fee_calculator_service.py`: Replaced `print()` with `logger.info()`.
        *   `app/services/ai_processor.py`: Confirmed `print()` already replaced.
        *   `app/services/document_comparison_service.py`: Added `logger` calls.
        *   `app/services/service_template.py`: Replaced `print()` with `logger` calls.
        *   `app/services/openai_service.py`: Replaced `print()` with `logger` calls.
        *   `app/services/precedent_service.py`: Replaced `print()` with `logger.info()`.
        *   `app/services/clause_library_service.py`: Replaced `print()` with `logger` calls.
        *   `app/services/client_intake_service.py`: Replaced numerous `print()` statements with various `logger` calls.
        *   `app/routes/contract_analysis_routes.py`: Replaced `print()` with `logger.info()` and `logger.error()`.
        *   `app/routes/legal_tools_routes.py`: Replaced `print()` with `logger.info()` and `logger.warning()`.

### Documentation

*   `[TODO] 2025-05-30`: **Create OpenAI Agents SDK Manual**:
    *   Objective: Develop a comprehensive, beginner-friendly Markdown document (`openai_agents_sdk_manual.md`) explaining the OpenAI Agents SDK.
    *   Action: Gather information from official SDK documentation, structure it logically (Introduction, Installation, Core Concepts, Tools, Handoffs, Tracing, Examples, Best Practices), and write clear explanations with code snippets.
*   `[TODO] 2025-05-30`: **Create/Update Project `README.md`**:
    *   Objective: Provide essential information for developers and users of the project.
    *   Action: Create or update the main `README.md` in the project root to include:
        *   Project description.
        *   Setup and installation instructions (including environment variables).
        *   How to run the server.
        *   Brief architectural overview.
        *   Link to API documentation (FastAPI's auto-generated docs).

### Testing
*   `[TODO] 2025-05-30`: **Develop Unit Testing Strategy for Services**:
    *   Objective: Ensure core business logic is reliable.
    *   Action: Plan and start writing unit tests for key services, beginning with `MemoryService` and `AIProcessor`.
    *   Requirement: Mock external dependencies (e.g., OpenAI API calls, database interactions once implemented).
*   `[TODO] 2025-05-30`: **Write Unit Tests for Logging**:
    *   Objective: Verify that logging calls are made correctly in services where `print` statements were replaced.
    *   Action: Start by writing unit tests for `OpenAIService`, `PrecedentService`, and `ClauseLibraryService`, mocking dependencies and asserting that logger methods are called with expected messages and levels.
*   `[TODO] 2025-05-30`: **Develop Integration Testing Strategy for API Endpoints**:
    *   Objective: Verify that API endpoints function correctly, including request validation, service interaction, and response generation.
    *   Action: Plan and start writing integration tests for critical API endpoints (e.g., memory storage/retrieval, AI response generation).

## Completed Tasks

*   `[DONE] 2025-06-01`: **Add/Improve Docstrings**:
    *   Objective: Ensure all public classes, methods, and functions have clear, informative docstrings.
    *   Action: Review and add/update docstrings in Google style for all modules in `app/services/`, `app/routes/`, `app/models/`, `app/config.py`, and other key files.
    *   **Progress (as of 2025-05-31):**
        *   **Services Completed:**
            *   `app/services/ai_processor.py`
            *   `app/services/citation_formatter_service.py`
            *   `app/services/memory_service.py`
            *   `app/services/openai_service.py`
            *   `app/services/clause_library_service.py`
            *   `app/services/client_intake_service.py`
            *   `app/services/contract_analysis_service.py`
            *   `app/services/predictive_analysis_service.py`
            *   `app/services/law_practice_service.py`
            *   `app/services/court_filing_service.py`
            *   `app/services/document_comparison_service.py`
            *   `app/services/document_template_service.py`
            *   `app/services/legal_fee_calculator_service.py`
            *   `app/services/legal_research_service.py`
            *   `app/services/precedent_service.py`
            *   `app/services/role_service.py`
            *   `app/services/service_template.py`
        *   **Routes Completed:**
            *   `app/routes/ai_processor_routes.py`
            *   `app/routes/clause_library_routes.py`
            *   `app/routes/client_intake_routes.py`
            *   `app/routes/contract_analysis_routes.py`
            *   `app/routes/document_template_routes.py`
            *   `app/routes/healthcheck.py`
            *   `app/routes/law_practice_routes.py`
            *   `app/routes/legal_tools_routes.py`
    *   **Progress (as of 2025-06-01):**
        *   **Models Completed:**
            *   `app/models/client_intake_models.py`
            *   `app/models/contract_analysis_models.py`
            *   `app/models/memory.py`
            *   `app/models/role.py`
        *   **Configuration Completed:**
            *   `app/config.py`
*   `[DONE] 2025-05-30`: **Refactor `AIProcessor` Error Handling and Logging**:
    *   Objective: Improved error handling in `AIProcessor` methods (`generate_response`, `create_embedding`, `process_prompt`) to use `HTTPException` and standardized logging.
    *   Actions Taken:
        *   Replaced `print` statements with `logger` calls (debug, info, warning, error, exception) throughout `ai_processor.py`.
        *   Modified error handling to catch specific OpenAI API errors (`AuthenticationError`, `RateLimitError`, `BadRequestError`, `APIError`) and re-raise them as `HTTPException`s with appropriate status codes (401, 429, 400, 502).
        *   General exceptions are caught and re-raised as `HTTPException(status_code=500, ...)` using `logger.exception` for stack traces.
        *   Implemented robust model selection fallback logic (explicit model -> task category -> instance default -> global default -> HTTPException) in `generate_response` and `process_prompt`.
        *   Removed redundant exception handling in `process_prompt`.
*   `[DONE] 2025-05-30`: **Implement Persistent Storage for `MemoryService` (Supabase - Code Complete)**:
    *   Objective: Replaced the in-memory storage in `MemoryService` with Supabase for persistence.
    *   Actions Taken:
        *   Added `supabase-py` to `requirements.txt`.
        *   Refactored `app/services/memory_service.py` to use Supabase client for CRUD operations, including a fallback for in-memory storage if Supabase is not configured.
        *   Defined `MemoryDB` model in `app/models/memory.py` aligned with the Supabase table schema and updated `MemoryCreate` and `Memory` models.
        *   Confirmed `app/config.py` includes necessary Supabase connection settings (`supabase_url`, `supabase_key`, `use_supabase`).
        *   Provided SQL schemas for the `memories` table and `match_memories` RPC function for Supabase.
    *   **USER ACTION REQUIRED**:
        1.  **Set up Supabase Project**:
            *   Enable the `vector` extension in your Supabase project dashboard (`Database` -> `Extensions`).
            *   Execute the provided SQL to create the `memories` table (ensure vector dimensions match your embedding model).
            *   Execute the provided SQL to create the `match_memories` function.
        2.  **Configure Environment Variables**: Update your `.env` file with your `SUPABASE_URL` and `SUPABASE_KEY`.
        3.  **Install Dependencies**: Run `pip install -r requirements.txt` in your virtual environment.
        4.  **Test Thoroughly**: Test all memory-related API endpoints to ensure the Supabase integration works as expected.
*   `[DONE] 2025-05-30`: **Review/Update `.env.example`**:
    *   Objective: Ensured the example environment file accurately reflects all required variables from the consolidated `app/config.py`.
    *   Action: Created `.env.example` in the project root with keys for `OPENAI_API_KEY`, `DEBUG`, `PORT`, `REDIS_URL`, `SUPABASE_URL`, and `SUPABASE_KEY`.
*   `[DONE] 2025-05-30`: **Consolidate Settings Files**:
    *   Objective: Established `app/config.py` as the single source of truth for all application settings.
    *   Action: Modified `AIProcessor`, `ClientIntakeService`, and `ContractAnalysisService` to use the `settings` instance derived from `app/config.py`.
    *   Action: Removed the redundant `app/settings.py` after ensuring all its necessary configurations were merged into `app/config.py` or handled by direct attribute access on the `app/config.Settings` instance.
*   `[DONE] 2025-05-30`: Initial review of Python MCP server structure (by Cascade).
*   `[DONE] 2025-05-30`: Creation of initial `PLANNING.md` and `TASK.md` (by Cascade).
