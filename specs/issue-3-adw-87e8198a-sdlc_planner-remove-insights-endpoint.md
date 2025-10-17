# Chore: Remove Insights Endpoint and Reorganize Imports

## Chore Description
Remove the `/api/insights` endpoint from the FastAPI server and move the `load_dotenv()` call to appear immediately after its import statement. This is a code cleanup task that will simplify the API surface and improve code organization. The insights endpoint is not currently used by the frontend application and can be safely removed along with its dependencies.

## Relevant Files
Use these files to resolve the chore:

- **`app/server/server.py`** - Main FastAPI server file containing the insights endpoint (lines 188-208) and the import/load_dotenv statements (lines 33-34). Need to remove the endpoint, the InsightsRequest/InsightsResponse imports (lines 16-17), and the generate_insights import (line 25), and move load_dotenv call after its import.

- **`app/server/core/data_models.py`** - Contains the data model definitions including InsightsRequest, InsightsResponse, and ColumnInsight. These models are only used by the insights endpoint and can be removed (lines 52-71).

- **`app/server/core/insights.py`** - Module that implements the generate_insights function. This entire file can be removed as it's only used by the insights endpoint.

- **`app/server/tests/test_sql_injection.py`** - Contains security tests for the insights module (line 22 import, lines 258-276). Need to remove the import and the TestInsightsModuleSecurity test class.

- **`app/client/src/api/client.ts`** - Frontend API client that defines generateInsights method (lines 64-73). This method should be removed as it's not used anywhere in the frontend.

- **`app/client/src/types.d.ts`** - TypeScript type definitions that likely include InsightsRequest and InsightsResponse types. Need to verify and remove if present.

- **`README.md`** - Documentation that lists `/api/insights` as an available endpoint (line 134). Need to remove this reference.

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### Step 1: Update server.py - Remove endpoint and reorganize imports
- Remove the InsightsRequest and InsightsResponse imports from line 16-17 in the data_models import
- Remove the generate_insights import from line 25
- Move the load_dotenv() call (currently line 34) to immediately follow the dotenv import (line 7), so it becomes line 8
- Remove the entire `/api/insights` endpoint function (lines 188-208)
- Verify no other code in server.py references insights functionality

### Step 2: Remove unused data models
- Delete the InsightsRequest model (lines 53-55 in core/data_models.py)
- Delete the ColumnInsight model (lines 57-66 in core/data_models.py)
- Delete the InsightsResponse model (lines 68-71 in core/data_models.py)

### Step 3: Remove insights module
- Delete the entire file `app/server/core/insights.py`

### Step 4: Update test file
- Remove the generate_insights import from line 22 in tests/test_sql_injection.py
- Remove the TestInsightsModuleSecurity test class (lines 258-276 in tests/test_sql_injection.py)

### Step 5: Update frontend API client
- Remove the generateInsights method from app/client/src/api/client.ts (lines 64-73)

### Step 6: Update frontend TypeScript types
- Open app/client/src/types.d.ts and check for InsightsRequest, InsightsResponse, or ColumnInsight type definitions
- Remove any insights-related type definitions if present

### Step 7: Update documentation
- Remove the `/api/insights` endpoint reference from the API Endpoints section in README.md (line 134)

### Step 8: Run validation commands
- Execute all validation commands listed below to ensure zero regressions

## Validation Commands
Execute every command to validate the chore is complete with zero regressions.

- `cd app/server && uv run ruff check .` - Lint check to ensure code quality
- `cd app/server && uv run pytest` - Run all server tests to validate no regressions
- `cd app/server && uv run python -m py_compile server.py` - Compile check for server.py
- `cd app/client && bun run type-check` - TypeScript type checking for frontend (if available)

## Notes
- The insights endpoint was defined but never actually used in the frontend application (verified by searching for generateInsights usage in frontend components)
- The insights functionality includes statistical analysis of table columns (unique values, null counts, min/max/avg for numeric columns, most common values)
- Removing this endpoint simplifies the API surface and reduces maintenance burden
- All insights-related code can be safely removed without breaking existing functionality
- The load_dotenv() reorganization is a minor code organization improvement to keep related imports and calls together
