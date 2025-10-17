# Bug: SQLite Test Running Against Live Database

## Bug Description
One of the SQLite tests in `app/server/tests/core/test_llm_processor.py` is running against the live database (`db/database.db`) and saving a table with the name pattern `data_...` (specifically `data___DROP_TABLE_users____`). This test should be using an in-memory SQLite database (`:memory:`) instead of the live database to prevent test pollution and data corruption.

The bug manifests as:
- Test data being written to the production database file
- Table `data___DROP_TABLE_users____` appearing in the live database
- Potential data corruption and test isolation issues
- Tests affecting the state of the live database

## Problem Statement
The test file `app/server/tests/core/test_llm_processor.py` does not mock the SQLite database connection properly. Unlike the other test files (`test_file_processor.py` and `test_sql_processor.py`) which use fixtures to create in-memory databases and patch the connection, the LLM processor tests are calling functions that indirectly use database operations without proper mocking, causing them to write to the live database at `db/database.db`.

## Solution Statement
Add proper database mocking to `app/server/tests/core/test_llm_processor.py` by creating a test fixture that provides an in-memory SQLite database and patches all relevant database connections. This will follow the same pattern used in `test_file_processor.py` and `test_sql_processor.py`, ensuring complete test isolation and preventing any writes to the live database.

## Steps to Reproduce
1. Navigate to `app/server/`
2. Check the live database tables: `python -c "import sqlite3; conn = sqlite3.connect('db/database.db'); cursor = conn.cursor(); cursor.execute('SELECT name FROM sqlite_master WHERE type=\"table\"'); print([row[0] for row in cursor.fetchall()])"`
3. Run the LLM processor tests: `uv run pytest tests/core/test_llm_processor.py -v`
4. Check the live database tables again - observe that new test tables have been created (e.g., `data___DROP_TABLE_users____`)
5. The test data persists in the live database after tests complete

## Root Cause Analysis
The root cause is located in `app/server/tests/core/test_llm_processor.py`:

1. **Missing Database Fixture**: The test file does not have a `test_db` fixture that creates an in-memory database
2. **No Connection Patching**: The tests do not patch `sqlite3.connect` calls in the modules under test
3. **Indirect Database Access**: The LLM processor functions are mocked, but any functions they might call that interact with the database are not properly isolated
4. **Inconsistent Test Patterns**: Unlike `test_file_processor.py:8-19` and `test_sql_processor.py:8-48` which both implement proper database mocking, `test_llm_processor.py` does not follow this pattern

The specific issue is that while the tests mock the LLM API calls (OpenAI and Anthropic), they don't prevent downstream database operations from occurring if the code paths were to execute actual database operations during testing.

## Relevant Files
Use these files to fix the bug:

- `app/server/tests/core/test_llm_processor.py` - The test file that needs proper database mocking added. This is the primary file to modify.
- `app/server/tests/core/test_file_processor.py:8-19` - Reference implementation showing how to create an in-memory test database fixture with proper connection patching.
- `app/server/tests/core/test_sql_processor.py:8-48` - Reference implementation showing the same pattern for database mocking in tests.
- `app/server/core/llm_processor.py` - The module being tested; review to ensure no direct database operations are being performed.
- `app/server/core/file_processor.py:58,130,290` - Shows where `sqlite3.connect("db/database.db")` is called, which needs to be mocked in tests.
- `app/server/core/sql_processor.py:18,66` - Shows where `sqlite3.connect("db/database.db")` is called, which needs to be mocked in tests.

### New Files
No new files need to be created for this bug fix.

## Step by Step Tasks
IMPORTANT: Execute every step in order, top to bottom.

### 1. Verify the Bug Exists
- Check the live database for test pollution: `cd app/server && python -c "import sqlite3; conn = sqlite3.connect('db/database.db'); cursor = conn.cursor(); cursor.execute('SELECT name FROM sqlite_master WHERE type=\"table\"'); print([row[0] for row in cursor.fetchall()])"`
- Document which tables are test-related (look for patterns like `data_...`)
- Run the LLM processor tests to confirm they run successfully: `cd app/server && uv run pytest tests/core/test_llm_processor.py -v`

### 2. Add Database Mocking Fixture to test_llm_processor.py
- Open `app/server/tests/core/test_llm_processor.py`
- Add a `test_db` fixture similar to the one in `test_file_processor.py:8-19` and `test_sql_processor.py:8-48`
- The fixture should:
  - Create an in-memory SQLite database using `sqlite3.connect(':memory:')`
  - Create any necessary test tables if LLM processor functions require them
  - Use `unittest.mock.patch` to mock all `sqlite3.connect` calls in the relevant modules
  - Properly yield the connection and close it after tests complete

### 3. Update Test Methods to Use the Fixture
- Review each test method in `TestLLMProcessor` class
- Add the `test_db` fixture as a parameter to any test that might indirectly cause database operations
- Ensure the fixture is properly applied before any test code executes

### 4. Clean Up the Live Database
- After fixing the tests, remove any test pollution from the live database
- Create a backup of the current database state: `cd app/server/db && cp database.db backup_before_cleanup.db`
- Remove test-related tables: `cd app/server && python -c "import sqlite3; conn = sqlite3.connect('db/database.db'); cursor = conn.cursor(); cursor.execute('DROP TABLE IF EXISTS data___DROP_TABLE_users____'); conn.commit()"`
- Verify cleanup: Check that only legitimate tables remain in the database

### 5. Run Validation Commands
- Execute all commands in the "Validation Commands" section below to validate the bug is fixed with zero regressions

## Validation Commands
Execute every command to validate the bug is fixed with zero regressions.

### Before Fix Verification
- `cd app/server && python -c "import sqlite3; conn = sqlite3.connect('db/database.db'); cursor = conn.cursor(); cursor.execute('SELECT name FROM sqlite_master WHERE type=\"table\"'); tables = [row[0] for row in cursor.fetchall()]; print('Tables before:', tables)"` - Document current database state

### Test Execution
- `cd app/server && uv run pytest tests/core/test_llm_processor.py -v` - Run LLM processor tests to verify they pass with the new fixture
- `cd app/server && uv run pytest tests/core/test_file_processor.py -v` - Verify file processor tests still pass
- `cd app/server && uv run pytest tests/core/test_sql_processor.py -v` - Verify SQL processor tests still pass
- `cd app/server && uv run pytest` - Run all server tests to validate zero regressions

### After Fix Verification
- `cd app/server && python -c "import sqlite3; conn = sqlite3.connect('db/database.db'); cursor = conn.cursor(); cursor.execute('SELECT name FROM sqlite_master WHERE type=\"table\"'); tables = [row[0] for row in cursor.fetchall()]; print('Tables after:', tables); test_tables = [t for t in tables if 'data_' in t.lower() or 'test' in t.lower() or 'hack' in t.lower()]; assert len(test_tables) == 0, f'Test pollution detected: {test_tables}'"` - Verify no test data is written to live database after running tests

### Build Verification
- `cd app/client && bun tsc --noEmit` - Run frontend type checking to validate zero regressions
- `cd app/client && bun run build` - Run frontend build to validate zero regressions

## Notes
- The bug was identified by finding the table `data___DROP_TABLE_users____` in the live database at `db/database.db`
- This appears to be related to SQL injection test cases that might have been inadvertently writing to the live database
- The fix follows the established pattern in the codebase where `test_file_processor.py` and `test_sql_processor.py` both use in-memory databases with proper mocking
- This is a critical bug because test data should never pollute the production database
- After this fix, all tests should be completely isolated and never touch the live database file
- The solution is surgical: we only modify `test_llm_processor.py` to add proper database mocking, matching the existing test patterns
