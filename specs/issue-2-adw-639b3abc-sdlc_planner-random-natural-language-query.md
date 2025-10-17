# Feature: Random Natural Language Query Generator

## Feature Description
This feature adds a new button to the UI that generates random, interesting natural language queries based on the existing database tables and their structure. When clicked, the button uses the LLM processor to create contextually relevant queries that demonstrate the capabilities of the natural language SQL interface. The generated query automatically populates the query input field, overwriting any existing content, allowing users to manually execute it. This feature helps users discover what types of questions they can ask and provides inspiration for querying their data.

## User Story
As a user
I want to generate random natural language queries based on my loaded tables
So that I can discover interesting insights and learn what types of questions I can ask about my data

## Problem Statement
Users often don't know what questions to ask about their data or how to phrase queries effectively. When faced with a blank query input field, users may struggle with:
- Understanding what types of questions are possible
- Learning the phrasing that works best with the natural language interface
- Discovering interesting patterns or relationships in their data
- Getting started with exploring their tables

A random query generator addresses this "blank slate" problem by providing concrete examples tailored to the user's actual data schema.

## Solution Statement
Implement a "Generate Random Query" button positioned alongside existing primary controls that:
1. Analyzes the current database schema (tables and columns)
2. Uses the existing LLM processor to generate contextually relevant, interesting natural language queries
3. Automatically populates the query input field with the generated query
4. Limits queries to two sentences maximum for clarity
5. Provides varied query types (aggregations, filters, joins, etc.) to showcase different capabilities

The solution leverages the existing `llm_processor.py` infrastructure and integrates seamlessly with the current UI design patterns.

## Relevant Files
Use these files to implement the feature:

- **app/server/core/llm_processor.py** - Contains the LLM integration functions. Will be extended to add a new function `generate_random_query()` that creates interesting queries based on schema information. This is the core backend logic for query generation.

- **app/server/server.py** - FastAPI server with all API endpoints. Will add a new `POST /api/generate-query` endpoint to handle random query generation requests from the frontend.

- **app/server/core/data_models.py** - Pydantic models for request/response validation. Will add `RandomQueryRequest` and `RandomQueryResponse` models to handle the new API endpoint.

- **app/server/core/sql_processor.py** - Contains `get_database_schema()` function that retrieves table and column information. Will be used to gather schema context for query generation.

- **app/client/src/main.ts** - Main TypeScript file containing UI initialization and event handlers. Will add button initialization and click handler for the random query generator.

- **app/client/src/api/client.ts** - API client functions for backend communication. Will add a new `generateRandomQuery()` method to call the new backend endpoint.

- **app/client/index.html** - HTML structure of the application. Will add the new "Generate Random Query" button to the query controls section.

- **app/client/src/style.css** - CSS styles for the application. Will add styles for the new button to match the Upload Data button style as specified.

- **app/client/src/types.d.ts** - TypeScript type definitions. Will add type definitions for `RandomQueryRequest` and `RandomQueryResponse`.

- **README.md** - Project documentation. Will update to document the new random query generation feature and its API endpoint.

### New Files

- **.claude/commands/e2e/test_random_query_generator.md** - E2E test specification for validating the random query generator feature works end-to-end in the browser.

- **app/server/tests/core/test_random_query_generator.py** - Unit tests for the random query generation backend logic, including schema parsing and query generation.

## Implementation Plan

### Phase 1: Foundation
Before implementing the main feature, we need to:
1. Understand the existing LLM processor architecture and prompt patterns
2. Review the schema retrieval mechanism to ensure we have all necessary context
3. Design the prompt engineering strategy for generating diverse, interesting queries
4. Define the API contract (request/response models)

### Phase 2: Core Implementation
The main implementation work involves:
1. **Backend Development**:
   - Create the random query generation logic in `llm_processor.py`
   - Implement prompt engineering to generate varied, contextually relevant queries
   - Add request/response models to `data_models.py`
   - Create the `/api/generate-query` endpoint in `server.py`
   - Add comprehensive error handling and validation

2. **Frontend Development**:
   - Add the "Generate Random Query" button to the UI
   - Style the button to match the "Upload Data" button aesthetic
   - Implement the click handler to call the API
   - Handle loading states and errors gracefully
   - Update the query input field with the generated query

### Phase 3: Integration
The feature integrates with existing functionality by:
1. Using the existing `get_database_schema()` function to retrieve table information
2. Leveraging the existing LLM processor infrastructure (OpenAI/Anthropic routing)
3. Populating the existing query input field with generated queries
4. Following the same error handling patterns as other API calls
5. Maintaining consistency with the existing UI/UX design patterns

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### Step 1: Add Backend Data Models
- Open `app/server/core/data_models.py`
- Add `RandomQueryRequest` model (empty or with optional parameters like `query_type`)
- Add `RandomQueryResponse` model with fields: `query` (str), `reasoning` (Optional[str]), `error` (Optional[str])
- Ensure models follow existing patterns in the file

### Step 2: Implement Random Query Generation Logic
- Open `app/server/core/llm_processor.py`
- Create a new function `generate_random_query(schema_info: Dict[str, Any]) -> str`
- Design a prompt that:
  - Takes the database schema as context
  - Requests interesting, varied natural language queries
  - Limits output to two sentences maximum
  - Encourages different query types (aggregations, filters, comparisons, joins)
- Use the existing routing logic to choose between OpenAI and Anthropic
- Handle cases where no tables exist (return helpful error)
- Add proper error handling and logging

### Step 3: Create Backend API Endpoint
- Open `app/server/server.py`
- Add a new `POST /api/generate-query` endpoint
- Use the `RandomQueryRequest` and `RandomQueryResponse` models
- Call `get_database_schema()` to retrieve current schema
- Call `generate_random_query()` with the schema
- Return the generated query in the response
- Add logging for success and error cases
- Follow existing endpoint patterns for consistency

### Step 4: Add Backend Unit Tests
- Create `app/server/tests/core/test_random_query_generator.py`
- Test `generate_random_query()` with various schema configurations:
  - Single table with numeric columns
  - Multiple tables for join queries
  - Empty database (no tables)
  - Tables with date/time columns
- Mock the LLM API calls to avoid external dependencies
- Verify query format and length constraints
- Test error handling

### Step 5: Add TypeScript Type Definitions
- Open `app/client/src/types.d.ts`
- Add `RandomQueryRequest` interface
- Add `RandomQueryResponse` interface
- Ensure types match the backend Pydantic models

### Step 6: Add API Client Method
- Open `app/client/src/api/client.ts`
- Add `generateRandomQuery()` method to the `api` object
- Follow existing patterns for API requests
- Return typed `RandomQueryResponse`

### Step 7: Add Button to HTML
- Open `app/client/index.html`
- Locate the `.query-controls` div (around line 22)
- Add a new button: `<button id="generate-query-button" class="secondary-button">Generate Random Query</button>`
- Position it after the "Upload Data" button to keep it visually separate from the primary "Query" button

### Step 8: Style the Button
- Open `app/client/src/style.css`
- Verify the `.secondary-button` class matches the "Upload Data" button style (already exists)
- Add any specific styles for `#generate-query-button` if needed for positioning or spacing
- Ensure consistent spacing with other buttons in `.query-controls`

### Step 9: Implement Button Functionality
- Open `app/client/src/main.ts`
- In `initializeQueryInput()` or create a new initialization function, add:
  - Get reference to `#generate-query-button`
  - Add click event listener
  - On click:
    - Disable button and show loading state
    - Call `api.generateRandomQuery()`
    - On success: populate `#query-input` textarea with the generated query (overwrite existing content)
    - On error: call `displayError()` with error message
    - Re-enable button and restore text
- Handle edge cases (no tables loaded, API errors, etc.)

### Step 10: Create E2E Test Specification
- Read `.claude/commands/test_e2e.md` to understand the E2E test format
- Read `.claude/commands/e2e/test_basic_query.md` as an example
- Create `.claude/commands/e2e/test_random_query_generator.md`
- Include test steps:
  1. Navigate to application
  2. Verify "Generate Random Query" button is present
  3. Load sample data (users table)
  4. Click "Generate Random Query" button
  5. Verify query input field is populated with generated query
  6. Verify query is non-empty and reasonable length (< 200 chars)
  7. Click the "Query" button to execute the generated query
  8. Verify results are displayed
  9. Take screenshots at key steps
- Define success criteria

### Step 11: Update Documentation
- Open `README.md`
- Add the new feature to the "Features" section
- Update "API Endpoints" section to include `POST /api/generate-query`
- Add usage notes in the "Usage" section explaining the random query generator

### Step 12: Manual Testing
- Start the server: `cd app/server && uv run python server.py`
- Start the client: `cd app/client && bun run dev`
- Open browser to `http://localhost:5173`
- Test scenarios:
  - Click "Generate Random Query" with no tables (should show error or helpful message)
  - Load sample data (users, products, events)
  - Click "Generate Random Query" multiple times
  - Verify queries are varied and interesting
  - Verify queries populate the input field (overwriting any existing content)
  - Execute generated queries and verify they work
  - Test error handling (stop server, click button)

### Step 13: Run All Validation Commands
- Execute all validation commands listed below to ensure zero regressions
- Fix any issues that arise
- Verify the feature works correctly end-to-end

## Testing Strategy

### Unit Tests
1. **Backend Query Generation (`test_random_query_generator.py`)**:
   - Test with single table schema
   - Test with multiple table schema (verify join queries)
   - Test with empty schema (no tables)
   - Test with various column types (numeric, text, date)
   - Test query length constraint (max 2 sentences)
   - Mock LLM API responses

2. **API Endpoint Testing**:
   - Test successful query generation
   - Test error handling (LLM API failure)
   - Test with empty database
   - Test response format validation

3. **Frontend Type Safety**:
   - Ensure TypeScript types match backend models
   - Test API client method with various responses

### Edge Cases
1. **No Tables Loaded**: Button should either be disabled or show a helpful error message when clicked
2. **LLM API Failure**: Graceful error handling with user-friendly message
3. **Very Large Schemas**: Ensure prompt doesn't exceed LLM token limits
4. **Special Characters in Table/Column Names**: Ensure proper escaping and formatting
5. **Slow LLM Response**: Loading state should be clear to user
6. **Multiple Rapid Clicks**: Button should be disabled during generation to prevent multiple concurrent requests
7. **Query Input Already Has Content**: New query should overwrite without warning (as specified)

## Acceptance Criteria
- [ ] "Generate Random Query" button is visible and styled consistently with "Upload Data" button
- [ ] Button is positioned in the query controls section, visually separate from the primary "Query" button
- [ ] Clicking the button generates a random, contextually relevant natural language query
- [ ] Generated queries are based on actual database schema (tables and columns)
- [ ] Generated queries are limited to two sentences maximum
- [ ] Generated query automatically populates the query input field, overwriting any existing content
- [ ] Button shows loading state while generating query
- [ ] Generated queries are varied (not identical on repeated clicks)
- [ ] Error handling works gracefully (no tables, API failure, etc.)
- [ ] Generated queries can be successfully executed by clicking the "Query" button
- [ ] Backend API endpoint `/api/generate-query` is documented and follows existing patterns
- [ ] Unit tests pass with 100% success rate
- [ ] Frontend TypeScript compilation succeeds with no errors
- [ ] E2E test validates the feature works in the browser
- [ ] README.md is updated with feature documentation

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

**Backend Validation:**
```bash
cd app/server && uv run pytest
```
Run all server tests including new random query generator tests.

**Frontend Type Check:**
```bash
cd app/client && bun tsc --noEmit
```
Ensure TypeScript types are correct and no compilation errors.

**Frontend Build:**
```bash
cd app/client && bun run build
```
Verify production build succeeds.

**E2E Test:**
1. Read `.claude/commands/test_e2e.md` to understand how to run E2E tests
2. Read and execute `.claude/commands/e2e/test_random_query_generator.md` to validate the feature works end-to-end in the browser
3. Verify all test steps pass and screenshots are captured

**Manual Validation:**
1. Start server and client
2. Load sample data (users, products, events)
3. Click "Generate Random Query" 5+ times
4. Verify queries are varied and interesting
5. Execute each generated query and verify results
6. Test error cases (no tables, server down)

## Notes

### LLM Prompt Design Considerations
The prompt for generating random queries should:
- Request specific query types to ensure variety: filters, aggregations, comparisons, time-based queries, multi-table joins
- Provide example query patterns to guide the LLM
- Emphasize generating queries that showcase interesting data insights
- Include the full schema with column names and types for context
- Use temperature > 0.3 (higher than SQL generation) to encourage creativity

### Future Enhancements
Consider these improvements for future iterations:
1. **Query Categories**: Allow users to choose query types (aggregation, filter, join, etc.)
2. **Query History**: Save generated queries for users to revisit
3. **Favoriting**: Let users mark interesting queries as favorites
4. **Query Complexity Slider**: Let users choose simple vs. complex queries
5. **Sample Results Preview**: Show what the query might return before executing
6. **Bulk Generation**: Generate multiple queries at once for users to browse

### Security Considerations
- Generated queries go through the same SQL injection protection as user-entered queries
- The LLM has access to schema information but not to actual data
- No additional security risks beyond existing query functionality

### Performance Notes
- Query generation requires an LLM API call (typically 1-3 seconds)
- Loading states should clearly indicate the generation is in progress
- Consider caching generated queries to reduce API calls (future enhancement)

### Accessibility
- Button should be keyboard accessible (standard button element)
- Loading state should be announced to screen readers
- Error messages should be clear and actionable
