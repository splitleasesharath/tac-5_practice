# Feature: Random Natural Language Query Button

## Feature Description
This feature adds a "Generate Random Query" button to the Natural Language SQL Interface that automatically generates interesting, contextually-aware natural language queries based on the database schema and table structures. When clicked, the button will use the LLM to create relevant query suggestions (limited to two sentences maximum) and populate the query input field, replacing any existing content. This provides users with example queries and helps them discover the capabilities of their data.

## User Story
As a user
I want to generate random natural language queries based on my database structure
So that I can explore my data, get query suggestions, and understand what questions I can ask

## Problem Statement
Users often face a blank query input field and don't know what questions to ask or what queries are possible with their uploaded data. This creates friction in the user experience and limits data exploration. Users need inspiration and examples of what types of queries they can ask based on their specific database schema.

## Solution Statement
Implement a new "Generate Random Query" button placed near the primary action buttons (Query and Upload Data) that uses the existing `llm_processor.py` infrastructure to analyze the current database schema and generate contextually-relevant natural language queries. The button will call a new backend endpoint that leverages the LLM to create interesting queries based on table structures, column types, and data characteristics. The generated query will be limited to two sentences maximum and will automatically populate (overwrite) the query input field.

## Relevant Files
Use these files to implement the feature:

- **Backend:**
  - `app/server/server.py` - Main FastAPI server where we'll add the new `/api/generate-query` endpoint
  - `app/server/core/llm_processor.py` - Contains existing LLM integration logic (OpenAI and Anthropic) that we'll extend with a new `generate_random_query()` function
  - `app/server/core/data_models.py` - Contains Pydantic models; we'll add `GenerateQueryRequest` and `GenerateQueryResponse` models
  - `app/server/core/sql_processor.py` - Contains `get_database_schema()` function which we'll use to provide schema context to the LLM

- **Frontend:**
  - `app/client/index.html` - HTML structure where we'll add the new "Generate Random Query" button in the query controls section
  - `app/client/src/main.ts` - Main TypeScript file where we'll add button event handler and populate the query input field
  - `app/client/src/api/client.ts` - API client where we'll add the `generateRandomQuery()` method
  - `app/client/src/types.d.ts` - TypeScript type definitions where we'll add the new API types
  - `app/client/src/style.css` - Styling where we'll add styles for the new button to match the "Upload Data" button style

- **Testing:**
  - `app/server/tests/core/test_llm_processor.py` - Existing LLM tests where we'll add tests for the new `generate_random_query()` function

### New Files
- `.claude/commands/e2e/test_random_query_button.md` - E2E test file to validate the random query generation feature works end-to-end

## Implementation Plan

### Phase 1: Foundation
First, we need to establish the data models and backend infrastructure. We'll extend the existing LLM processor with a new function that analyzes database schemas and generates contextually-aware natural language queries. This foundation ensures we can properly communicate between frontend and backend, and that the LLM has the right context to generate meaningful queries.

### Phase 2: Core Implementation
Next, we'll implement the core functionality including the FastAPI endpoint that calls the LLM with schema context, and the frontend button that triggers this endpoint and populates the input field. This phase brings the feature to life by connecting all the pieces together and ensuring the query generation actually works.

### Phase 3: Integration
Finally, we'll integrate the new button into the existing UI, ensure it matches the design patterns (following the "Upload Data" button style), add comprehensive error handling, and create E2E tests to validate the entire flow works correctly from user interaction to query population.

## Step by Step Tasks

### Step 1: Create Backend Data Models
- Open `app/server/core/data_models.py`
- Add `GenerateQueryRequest` Pydantic model (empty model, no parameters needed)
- Add `GenerateQueryResponse` Pydantic model with fields: `query` (str), `context` (Optional[str]), `error` (Optional[str])
- These models ensure type safety and validation for the new API endpoint

### Step 2: Extend LLM Processor with Query Generation
- Open `app/server/core/llm_processor.py`
- Create new function `generate_random_query(schema_info: Dict[str, Any], provider: str = "openai") -> str`
- Implement logic to format schema information into a prompt that asks the LLM to generate an interesting natural language query based on the tables and columns
- The prompt should instruct the LLM to:
  - Generate a query that explores interesting aspects of the data
  - Use table and column names from the schema
  - Create queries that demonstrate different SQL capabilities (filtering, aggregation, joins, etc.)
  - Limit the output to two sentences maximum
  - Return only the natural language query text, no SQL
- Support both OpenAI and Anthropic providers using existing helper functions
- Add proper error handling and logging

### Step 3: Create Backend API Endpoint
- Open `app/server/server.py`
- Add new POST endpoint `/api/generate-query` that returns `GenerateQueryResponse`
- The endpoint should:
  - Call `get_database_schema()` to get current schema
  - Check if there are any tables in the database (return error if empty)
  - Call `generate_random_query()` with the schema information
  - Return the generated query in the response
  - Handle errors gracefully and return appropriate error messages
- Add logging for successful generation and errors

### Step 4: Add Backend Tests
- Open `app/server/tests/core/test_llm_processor.py`
- Add test function `test_generate_random_query_with_openai()` that mocks the OpenAI API
- Add test function `test_generate_random_query_with_anthropic()` that mocks the Anthropic API
- Add test function `test_generate_random_query_empty_schema()` that validates error handling
- Ensure tests validate query length constraints (two sentences max)
- Run tests with `cd app/server && uv run pytest tests/core/test_llm_processor.py -v`

### Step 5: Add Frontend TypeScript Types
- Open `app/client/src/types.d.ts`
- Add `GenerateQueryRequest` interface (empty interface)
- Add `GenerateQueryResponse` interface with fields: `query`, `context?`, `error?`
- Ensure types match the backend Pydantic models exactly

### Step 6: Extend Frontend API Client
- Open `app/client/src/api/client.ts`
- Add new method `generateRandomQuery(): Promise<GenerateQueryResponse>` to the `api` object
- Use the existing `apiRequest` helper to make POST request to `/generate-query`
- Return properly typed response

### Step 7: Add Button to HTML
- Open `app/client/index.html`
- In the `.query-controls` div (around line 22-25), add a new button after the Upload Data button
- Add button with id `generate-query-button`, class `secondary-button`, and text "Generate Random Query"
- Position the button so it's visually separated from the primary buttons but still accessible

### Step 8: Implement Frontend Button Logic
- Open `app/client/src/main.ts`
- Create new function `initializeGenerateQuery()` that:
  - Gets references to the generate query button and query input field
  - Adds click event listener to the button
  - On click, disables the button and shows loading state
  - Calls `api.generateRandomQuery()`
  - On success, overwrites the query input field value with the generated query
  - On error, displays error message using existing `displayError()` function
  - Re-enables button after completion
- Call `initializeGenerateQuery()` from the `DOMContentLoaded` event listener (around line 7-12)

### Step 9: Style the New Button
- Open `app/client/src/style.css`
- Add styles for the `.generate-query-button` to match the "Upload Data" button style (secondary-button class)
- Ensure proper spacing and alignment with other buttons in the query controls section
- Add hover and active states for better user feedback

### Step 10: Create E2E Test File
- Read `.claude/commands/test_e2e.md` and `.claude/commands/e2e/test_basic_query.md` to understand the E2E test format
- Create new file `.claude/commands/e2e/test_random_query_button.md`
- Define test steps that:
  1. Navigate to the application
  2. Upload sample data (users.json)
  3. Verify the "Generate Random Query" button is visible
  4. Click the "Generate Random Query" button
  5. Verify the query input field is populated with text
  6. Verify the query text is relevant to the uploaded data (mentions table/column names)
  7. Verify the query is not empty and is limited to reasonable length
  8. Take screenshots at each major step
  9. Optionally: Click the Query button to execute the generated query and verify it works
- Include success criteria that validate button functionality, query population, and query relevance

### Step 11: Run All Validation Commands
- Execute all commands in the "Validation Commands" section to ensure zero regressions
- Fix any issues that arise during validation
- Ensure all tests pass before marking the feature as complete

## Testing Strategy

### Unit Tests
- **Backend LLM Processor Tests:**
  - Test `generate_random_query()` with mocked OpenAI API responses
  - Test `generate_random_query()` with mocked Anthropic API responses
  - Test error handling when no tables exist in database
  - Test that generated queries are limited to two sentences maximum
  - Test provider routing logic (OpenAI priority, then Anthropic fallback)

- **Backend API Endpoint Tests:**
  - Test `/api/generate-query` returns valid query when tables exist
  - Test `/api/generate-query` returns error when no tables exist
  - Test proper error handling when LLM API fails

### Integration Tests
- **E2E Browser Tests:**
  - Test button visibility and accessibility
  - Test button click triggers API call
  - Test query input field is populated (overwritten) with generated query
  - Test loading state is shown during generation
  - Test error handling displays errors correctly
  - Test generated query can be executed successfully
  - Test button works multiple times (can generate different queries)

### Edge Cases
- **No tables in database:** Button should handle gracefully, show appropriate error message
- **Very complex schema:** Ensure LLM can handle schemas with many tables and columns without errors
- **API failures:** Test both OpenAI and Anthropic API failure scenarios
- **Empty query generation:** Handle case where LLM returns empty or invalid response
- **Existing text in input field:** Verify the new query overwrites existing text as specified
- **Rapid clicking:** Ensure button is properly disabled during generation to prevent duplicate requests
- **Long generated queries:** Verify two-sentence limit is enforced
- **Network errors:** Handle timeout and connection failures gracefully

## Acceptance Criteria
- [ ] A "Generate Random Query" button is visible in the UI, styled consistently with the "Upload Data" button
- [ ] Button is positioned appropriately, visually separated from primary action buttons
- [ ] Clicking the button triggers a call to the new `/api/generate-query` endpoint
- [ ] The endpoint uses `llm_processor.py` to generate queries based on current database schema
- [ ] Generated queries are contextually relevant to the uploaded tables and columns
- [ ] Generated queries are limited to two sentences maximum
- [ ] The query input field is populated (overwritten) with the generated query
- [ ] Button shows loading state during generation
- [ ] Errors are handled gracefully with user-friendly messages
- [ ] Generated queries can be executed successfully using the existing Query button
- [ ] All existing functionality remains unaffected (zero regressions)
- [ ] Backend unit tests pass with coverage for new functionality
- [ ] E2E test validates the complete user flow
- [ ] Code follows existing patterns and conventions in the codebase

## Validation Commands
Execute every command to validate the feature works correctly with zero regressions.

- `cd app/server && uv run pytest` - Run server tests to validate the feature works with zero regressions
- `cd app/server && uv run pytest tests/core/test_llm_processor.py -v` - Run LLM processor tests specifically to validate new query generation function
- `cd app/client && bun run tsc --noEmit` - Run frontend TypeScript compilation to validate type safety
- `cd app/client && bun run build` - Run frontend build to validate the feature works with zero regressions
- Read `.claude/commands/test_e2e.md`, then read and execute the new E2E test file `.claude/commands/e2e/test_random_query_button.md` to validate this functionality works end-to-end

## Notes

### Design Decisions
- **Button placement:** The button is placed alongside the Upload Data button to keep all data-related actions grouped together, but visually separated from the primary Query action to avoid confusion
- **Overwrite behavior:** The feature explicitly overwrites the query input field rather than appending, ensuring a clean slate for the generated query
- **Two-sentence limit:** This constraint keeps queries concise and readable while still allowing for meaningful exploration
- **LLM provider routing:** The feature reuses the existing provider routing logic (OpenAI priority, then Anthropic) to maintain consistency with the rest of the application

### Future Considerations
- **Query history:** Could extend this feature to remember previously generated queries and avoid duplicates
- **Query categories:** Could add options to generate specific types of queries (aggregations, filters, joins, etc.)
- **Multiple suggestions:** Could generate 3-5 queries at once and let users choose
- **Favorites:** Could allow users to save interesting generated queries for later use
- **Schema-aware suggestions:** Could use table relationships and foreign keys to generate more sophisticated join queries
- **Sample data analysis:** Could analyze actual data values (not just schema) to generate more relevant queries

### Technical Notes
- The feature uses the existing `get_database_schema()` function which provides table names, column names, column types, and row counts
- The LLM prompt should be carefully crafted to generate diverse query types (simple selects, aggregations, filtering, ordering, etc.)
- Error handling is critical since LLM APIs can fail or return unexpected responses
- The frontend should provide immediate visual feedback (loading state) since LLM calls can take 1-3 seconds
- Consider rate limiting if this feature is used heavily to avoid excessive API costs

### Dependencies
- No new dependencies required - uses existing OpenAI and Anthropic SDK installations
- Feature leverages existing infrastructure in `llm_processor.py` for consistency
