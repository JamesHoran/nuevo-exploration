Phase 1: MVP - Core Gameplay & Learning Loop

This phase focuses on building the minimum set of features required to validate the core user experience: playing through a story chapter and engaging with the basic SRS.

    [ ] 1. Implement MVP Database Schema

        Description: Populate the prisma/schema.prisma file with the models required for the MVP: User, Story, Chapter, Scene, Choice, ChoiceOutcome, PlayerState, UserChoice, VocabularyItem, and SrsState.

        Testing & Verification:

            Copy the relevant models from the architecture document (Section 2.2) into schema.prisma.

            Run npx prisma format to ensure correct syntax.

            Run npx prisma migrate dev --name "initial-mvp-schema" to apply the schema to the database.

        Success Criteria: The migration completes successfully. The database tables are created and can be viewed using Prisma Studio (npx prisma studio).

    [ ] 2. Implement SRS Algorithm & Unit Test

        Description: Create the applySm2 function in src/lib/srs.ts exactly as specified in the architecture (Section 3.1). Write comprehensive unit tests to validate its logic against the SM-2 specification.

        Testing & Verification:

            Create the file src/lib/srs.ts and add the applySm2 function.

            Create a corresponding test file src/lib/srs.test.ts.

            Write Jest unit tests for all quality inputs (0-5) and for different repetition states (0, 1, and >1).

            Assert that repetitions, intervalDays, efactor, and nextReviewDate are calculated correctly in each case.

        Success Criteria: All unit tests for the applySm2 function pass.

    [ ] 3. Build Core API Endpoints

        Description: Create the Next.js Route Handlers for the MVP's core actions: fetching scene data, submitting a choice, and recording an SRS review. Implement basic user authentication to protect these endpoints.

        Testing & Verification:

            Create /api/story/scene/[sceneId]/route.ts: Fetches sceneData from the database.

            Create /api/story/choice/route.ts: Receives a choiceId, validates it, updates PlayerState, logs the UserChoice, and returns the nextSceneId.

            Create /api/learning/review/route.ts: Receives a vocabId and quality, calls the applySm2 function, and updates the SrsState in the database.

            Use an authentication library (e.g., NextAuth.js) to protect these routes.

            Write integration tests for each endpoint using Jest and a test database to verify database interactions.

        Success Criteria: API endpoints function as expected. Protected endpoints return a 401/403 error without authentication and a 200/201 status with valid authentication.

    [ ] 4. Develop Core Game UI Components

        Description: Build the React components necessary to render the game interface: SceneRenderer, DialogueBox, ChoiceList, and VocabularyTooltip.

        Testing & Verification:

            Create each component in src/components/game/.

            Use Storybook or a simple test page to develop components in isolation, passing them mock data that matches the sceneData JSON structure.

            The SceneRenderer should correctly parse the sceneData and render the other components.

            The VocabularyTooltip should appear on hover/tap of a vocab word and display its translation.

        Success Criteria: Components render correctly with mock data. Interactions (like clicking a choice) trigger the appropriate callback functions.

    [ ] 5. Implement Client-Side State & Data Fetching

        Description: Use React Query to fetch scene data from the API and manage server state. Use Zustand for minimal global UI state like audio settings.

        Testing & Verification:

            On the main game page (/story/[storyId]/[chapterId]/page.tsx), use a React Query useQuery hook to fetch the initial scene data.

            Use a useMutation hook to handle submitting a choice to the /api/story/choice endpoint. On success, the mutation should invalidate the scene query to fetch the next scene.

            Create a simple Zustand store for a global setting like isMuted.

        Success Criteria: The game page loads the first scene. Making a choice successfully sends a request to the backend and the UI updates to show the next scene.

    [ ] 6. Create Standalone SRS Review UI

        Description: Build a simple, non-narrative page where users can review vocabulary items due for study.

        Testing & Verification:

            Create an API endpoint /api/learning/reviews that fetches all SrsState records for the user where nextReviewDate is in the past.

            Create a page at /review that uses React Query to fetch this list.

            The UI should display one card at a time, with buttons for the user to rate their recall quality (e.g., "Forgot," "Hard," "Good," "Easy").

            Clicking a button should trigger a mutation to the /api/learning/review endpoint.

        Success Criteria: The review page correctly displays due cards. Submitting a review updates the card's state and removes it from the current session.

    [ ] 7. Seed Database with MVP Content

        Description: Manually create the data for one complete, playable story chapter. This includes the Story, Chapter, Scene (with sceneData JSON), Choice, ChoiceOutcome, and VocabularyItem records.

        Testing & Verification:

            Write a Prisma seed script (prisma/seed.ts) to populate the database with the content for the first chapter.

            Ensure choices correctly link scenes to create a valid, branching path.

            Tag several VocabularyItems within the sceneData JSON.

        Success Criteria: Running npx prisma db seed populates the database. The data is sufficient to play through the entire first chapter from start to finish.

    [ ] 8. Write MVP E2E Tests

        Description: Create Playwright E2E tests for the most critical user flows of the MVP.

        Testing & Verification:

            Golden Path Test: A test that logs in a user, starts the first chapter, navigates through a specific path of choices to the end, and logs out.

            SRS Loop Test: A test that plays the game to encounter a new word, navigates to the review page, successfully reviews that word, and verifies (by checking the UI or making an API call) that the word is no longer due for review.

        Success Criteria: All E2E tests pass reliably in a headless browser environment.
