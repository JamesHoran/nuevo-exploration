This final development phase transforms the application from a solid product into a polished, feature-rich experience. The focus is on implementing advanced gameplay systems, full offline capability, and social features that encourage long-term retention.

    [ ] 1. Implement Advanced RPG Mechanics (Stats & Inventory)

        Description: Expand the core gameplay engine to include deeper RPG elements. This involves adding character stats to the PlayerState model and implementing the logic for stat checks and inventory-based puzzles within the narrative.

        Testing & Verification:

            Update the PlayerState model in schema.prisma to include a JSON field for stats (e.g., {"charisma": 5, "wisdom": 7}). Add the Achievement and UserAchievement models. Run a database migration.

            Enhance the /api/story/choice endpoint's logic to parse the conditions JSON on a ChoiceOutcome. It must now check the player's PlayerState for required stats or inventory items before allowing a choice.

            Update the logic to correctly apply stateChanges that grant items, modify stats, or award achievements.

            Write specific integration tests for the choice endpoint: one where a choice is permitted due to correct stats, one where it's denied, and one where making a choice consumes an inventory item.

            Update the admin story editor to allow authors to easily define these conditions and state changes (e.g., dropdowns for "requires item" or "stat check > 10").

        Success Criteria: The narrative can now present choices that are conditionally available based on a player's stats or inventory. The game feels more like a true RPG, where player development unlocks new story paths.

    [ ] 2. Implement Full PWA & Offline-First Strategy

        Description: Convert the application into a full-fledged Progressive Web App (PWA) with a robust offline-first strategy. This ensures a seamless, native-like experience on mobile devices, even with intermittent connectivity.

        Testing & Verification:

            Configure the project with a manifest.json file and set up a service worker using a library like serwist to manage caching.

Implement the "chapter-aware prefetching" logic. When a user starts a chapter online, the service worker should be triggered to download and cache all necessary assets for that chapter (scene data, images, audio).

Use IndexedDB to store a queue of actions performed while offline (e.g., choices made, exposures logged).

Implement a background sync mechanism that sends the queued actions to the server once connectivity is restored.

        Use Playwright's network throttling and offline simulation features to write an E2E test:

            The test starts a chapter online.

            Goes offline.

            Successfully navigates several scenes.

            Goes back online.

            The test verifies that the progress made while offline is correctly reflected in the database.

    Success Criteria: A user can add the app to their home screen, play through an entire pre-cached chapter without an internet connection, and have their progress sync automatically upon reconnecting.

[ ] 3. Build Social Engagement Features (Leaderboards)

    Description: Introduce social features to increase motivation and long-term engagement. The first feature will be a leaderboard system based on player XP and vocabulary mastery.

    Testing & Verification:

        Create a new API endpoint (e.g., /api/leaderboards/xp) that performs an aggregate query on the database to return a ranked list of users by their PlayerState.xp. Ensure the query is indexed and performant.

        Build a new page and corresponding React components at /leaderboard to fetch and display this data.

        Implement UI for different leaderboard types (e.g., weekly XP, total vocabulary mastered).

        Write an E2E test that performs an in-game action to earn XP, then navigates to the leaderboard page and asserts that the user's rank and score have been updated correctly.

    Success Criteria: Players can view their ranking relative to others, fostering a sense of friendly competition and community.

[ ] 4. Implement Analytics & Final Polish

    Description: Integrate a basic, privacy-conscious analytics system to gather insights for future development. Conduct a final round of polish across the entire application, focusing on UI consistency, performance, and accessibility.

    Testing & Verification:

        Add the AnalyticsEvent model to the schema and create the /api/analytics/event endpoint for logging key user actions.

        On the client, create a simple analytics service that sends events for important interactions (chapter_started, choice_made, review_failed, etc.).

        Conduct a thorough manual QA pass of the entire application, specifically testing for:

            UI/UX: Consistent design language, smooth animations (using Framer Motion), and intuitive user flows.

            Accessibility: Full keyboard navigability, proper ARIA roles for all interactive elements, and sufficient color contrast.

            Performance: Use Next.js Bundle Analyzer to check for oversized client components and Lighthouse scores to measure Core Web Vitals.

    Success Criteria: The application is now tracking core events to inform data-driven decisions. The final product is polished, performant, accessible, and ready for a public launch.
