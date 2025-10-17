# E2E Test: Random Query Generation Button

Test the random query generation functionality in the Natural Language SQL Interface application.

## User Story

As a user
I want to generate random natural language queries based on my database structure
So that I can explore my data, get query suggestions, and understand what questions I can ask

## Test Steps

1. Navigate to the `Application URL`
2. Take a screenshot of the initial state
3. **Verify** the page title is "Natural Language SQL Interface"
4. **Verify** core UI elements are present:
   - Query input textbox
   - Query button
   - Upload Data button
   - Generate Random Query button
   - Available Tables section

5. **Verify** the "Generate Random Query" button is visible and enabled
6. Take a screenshot showing the Generate Random Query button

7. Upload sample data (users.json) by:
   - Clicking the "Upload Data" button
   - Clicking on the "Users Data" sample button in the modal
8. Wait for upload to complete
9. **Verify** the users table appears in the Available Tables section
10. Take a screenshot of the tables section with uploaded data

11. Click the "Generate Random Query" button
12. **Verify** the button shows a loading state ("Generating...")
13. Wait for the query generation to complete
14. **Verify** the query input field is populated with text
15. **Verify** the generated query is not empty
16. **Verify** the generated query mentions "users" (the table name) or column names from the users table
17. Take a screenshot of the populated query input field

18. Click the "Generate Random Query" button again
19. Wait for completion
20. **Verify** a new query is generated (field is populated)
21. Take a screenshot of the second generated query

22. Click the "Query" button to execute the generated query
23. Wait for query execution
24. **Verify** query results are displayed
25. **Verify** no errors are shown
26. Take a screenshot of the query results

## Success Criteria
- Generate Random Query button is visible and accessible
- Button shows loading state during generation
- Query input field is populated with generated query text
- Generated query is relevant to the uploaded data (mentions table/column names)
- Generated query is not empty and has reasonable length
- Button can be clicked multiple times to generate different queries
- Generated queries can be executed successfully
- 6 screenshots are taken
