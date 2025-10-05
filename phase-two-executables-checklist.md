Phase 2: V1 - Content Pipelines & Enhanced Learning

With the core gameplay loop validated, this phase introduces the automated content pipelines and advanced learning features that will enable the application to scale. The focus is on empowering content creators and making the learning experience more immersive and effective.

    [ ] 1. Set Up Asynchronous Job Infrastructure

        Description: Configure the foundational infrastructure for all background processing. This includes setting up a Redis instance for the job queue, an S3-compatible object storage bucket for assets, and the basic scaffolding for background worker functions.

        Testing & Verification:

            Provision a Redis instance (e.g., Upstash, Vercel KV) and an S3-compatible bucket (e.g., AWS S3, Cloudflare R2). Add credentials to the project's environment variables.

            Create a lib/queue.ts module to abstract pushing jobs to Redis.

            Create a simple test worker (e.g., /api/workers/test-worker) that can be triggered by a job.

            Write an integration test that pushes a "ping" job to the queue and polls the worker's logs or a database flag to confirm it was executed.

        Success Criteria: A job can be successfully enqueued from the main application, and a background worker consumes and processes it. The connection to S3 is verified by successfully uploading and then retrieving a test file.

    [ ] 2. Implement Image Generation Pipeline (Backend)

        Description: Build the complete backend workflow for asynchronous image generation. This involves creating the necessary database models, the API endpoint for admins to initiate jobs, and the background worker to execute them.

        Testing & Verification:

            Add the ImageGenJob and Asset models to schema.prisma and run a migration (npx prisma migrate dev).

            Create the admin-only API endpoint /api/jobs/image-gen (POST) that creates an ImageGenJob record and enqueues a job.

            Create the image generation worker. This worker will:

                Dequeue the job.

                Update the job status to PROCESSING.

                Call a third-party image generation API (e.g., Stability AI).

                On success, upload the image to the S3 bucket.

                Create a new Asset record with the S3 URL and a PENDING status.

                Update the ImageGenJob to COMPLETED and link it to the new Asset.

            Write an integration test for the API endpoint. Manually trigger a job and verify the image appears in S3 and all database records are correctly created and updated.

        Success Criteria: A POST request to the API endpoint with a prompt results in a new image file in the S3 bucket and a corresponding Asset record in the database with a PENDING status.

    [ ] 3. Build Image Generation Admin UI

        Description: Create the user interface within the (admin) section for content creators to manage the entire image generation process.

        Testing & Verification:

            Build a new page at /admin/image-generator.

            The UI should include a form to submit a new prompt, which calls the API from the previous step.

            Add a section that polls and displays the status of recent ImageGenJobs.

            Create an "Approval Queue" page at /admin/approval-queue that displays all Assets with a PENDING status.

            The queue UI must have "Approve" and "Reject" buttons for each asset, which trigger API calls to update the asset's status in the database.

            Manually test the full flow: an admin generates an image, sees it in the queue, and successfully approves it.

        Success Criteria: An admin can manage the entire image lifecycle—from prompt to approved asset—without leaving the admin panel.

    [ ] 4. Implement Text-to-Speech (TTS) Pipeline

        Description: Develop an automated, pre-generation pipeline for creating narration audio. This process is triggered when a chapter is marked as "ready for publishing."

        Testing & Verification:

            Add a "Publish" button to the admin story editor UI.

            Clicking "Publish" should trigger an API call that enqueues a tts-generation job with the chapterId.

            Create a TTS worker that:

                Fetches all scenes and dialogue lines for the given chapter.

                For each line, calls a TTS API (e.g., ElevenLabs, Azure Speech Service).

            Saves the returned audio file to S3 with a predictable name (e.g., audio/scene_{id}/line_{id}.mp3).

            Creates an Asset record for the audio file with an APPROVED status.

            Updates the sceneData JSON for the corresponding scene to include the audioAssetId.

        Manually test by publishing a chapter with a few lines of dialogue. Verify the audio files exist in S3 and the sceneData in the database is correctly updated with the asset IDs.

    Success Criteria: Publishing a chapter automatically generates and links all narration audio, making the chapter fully voiced.

[ ] 5. Implement ASR Pronunciation Scoring

    Description: Integrate a server-side Automatic Speech Recognition (ASR) service to provide detailed pronunciation feedback to users, which then influences their SRS state.

    Testing & Verification:

        In the VocabularyTooltip or a dedicated practice UI, add a "Practice Pronunciation" button that uses the browser's MediaRecorder API to capture audio.

        Create the /api/learning/asr endpoint that accepts a POST request with an audio blob and a vocabId.

        The endpoint's handler should send the audio to a specialized ASR service (e.g., Speechace) that returns detailed scoring, including a phoneme-level analysis.

        The handler then normalizes the overall score to a 0-5 quality rating, calls the applySm2 function, and updates the user's SrsState for that vocabId.

        The detailed feedback (e.g., mispronounced phonemes) is returned to the client to be displayed to the user.

        Write an E2E test using Playwright that mocks the MediaRecorder API to provide a pre-recorded audio file and verifies that the UI displays the feedback returned from the (mocked) ASR service.

    Success Criteria: A user can record themselves saying a word, receive actionable pronunciation feedback, and have their performance directly and correctly impact their learning schedule.

[ ] 6. Integrate Diegetic SRS Reviews into Narrative

    Description: Weave SRS review activities directly into the gameplay at designated points in the story, making review a core game mechanic rather than a separate mode.

    Testing & Verification:

        In the admin Scene Editor, add a checkbox or flag to designate a scene as a "review point."

        Modify the /api/story/scene/[sceneId] endpoint. If a scene is a review point, the handler should first query for any due SrsState items for the current user.

        If a due item is found, the handler should dynamically alter the sceneData JSON before returning it. For example, it could inject a new interactiveHotspot that requires the user to type the due vocabulary word to proceed.

        Create a test user and use Prisma Studio to manually set a vocabulary item's nextReviewDate to yesterday.

        Write a Playwright test where this user navigates to the designated review point scene. Assert that the special review puzzle is present. The test should then solve the puzzle and verify that the story continues.

    Success Criteria: A player's review queue directly impacts their gameplay, with due items appearing as contextual challenges that must be overcome to advance the narrative.

[ ] 7. Write V1 E2E Tests

    Description: Enhance the Playwright E2E suite to cover the new, complex flows introduced in V1, ensuring the integrated system works as expected.

    Testing & Verification:

        Full Content Pipeline Test: Write a test where an admin user logs in, generates and approves an image, creates a new scene using that image, publishes the chapter to generate TTS audio, and then a player user plays through that scene, verifying the correct image and audio are loaded.

        ASR-SRS Loop Test: Write a test where a user has a word with a known nextReviewDate. The user then completes an ASR practice for that word with a high score. The test then queries the database (via a helper API endpoint for testing) to assert that the new nextReviewDate is further in the future than the original date.

    Success Criteria: The critical paths for content creation and advanced learning mechanics are covered by automated tests, ensuring stability and preventing regressions.
