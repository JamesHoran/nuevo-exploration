# **A Complete Architectural Blueprint for a "ClueFinders-Style" Spanish Learning RPG**

## **I. Executive Blueprint: Vision & System Architecture**

This section establishes the foundational technical strategy for the Spanish language learning RPG. It presents a comprehensive overview of the system's architecture, from the high-level interaction of its components to the specific technologies chosen for implementation. The design prioritizes scalability, developer experience, and a seamless integration of gameplay and pedagogical mechanics, all while adhering to modern best practices for production-ready web applications.

### **1.1 High-Level Architecture**

The proposed architecture is a "scalable monolith" built upon a serverless-native foundation, leveraging the full capabilities of the Next.js App Router for both the client application and the API layer.1 This approach maximizes development velocity by unifying the codebase, while retaining the scalability benefits of serverless infrastructure. The system is designed to be deployed on Vercel, whose framework-aware platform provides deep integration and performance optimizations for Next.js applications.3  
The flow of data and requests between the system's components is illustrated below. This design decouples synchronous user interactions from asynchronous, computationally intensive tasks, ensuring a responsive user experience and robust backend processing.  
\+---------------------------------+ \+---------------------------+ \+---------------------------------+  
| Client (Browser / PWA) | | Edge / CDN (Vercel) | | API Layer (Next.js) |  
|---------------------------------| |---------------------------| |---------------------------------|  
| \- Next.js App Router (React) |\<----\>| \- Caching (Static Assets, |\<----\>| \- Route Handlers (Serverless) |  
| \- TypeScript | | Images, Audio, Data) | | \- Authentication & Authorization|  
| \- Zustand / React Query | | \- Image Optimization | | \- Synchronous Logic |  
| \- Howler.js / Web Audio API | | \- Edge Middleware | | (Fetch Scene, Submit Choice) |  
\+---------------------------------+ \+---------------------------+ \+---------------------------------+  
|  
| (Pushes Jobs)  
v  
\+---------------------------------+ \+---------------------------+ \+---------------------------------+  
| Background Workers | | Queue (Redis) | | Database (PostgreSQL) |  
| (Vercel Functions / AWS Lambda) | |---------------------------| |---------------------------------|  
|---------------------------------| | \- Upstash / Vercel KV |\<----\>| \- Prisma ORM |  
| \- Image Generation (AI) |\<----\>| \- Job Decoupling | | \- User Data & Progress |  
| \- Audio Processing (TTS/ASR) | | \- Rate Limiting | | \- Story & Content Data |  
| \- Bulk SRS Computations | | \- Retry Management | | \- SRS State |  
\+---------------------------------+ \+---------------------------+ \+---------------------------------+  
^ | | ^  
| | | (Read/Write Assets) | (Read/Write State)  
| | \+------------------------\>+---------------------------+ |  
| | | Asset Storage (S3-like) |\<----+  
| | |---------------------------|  
| | | \- Cloudflare R2 / AWS S3 |  
| | | \- Generated Images |  
| | | \- Generated Audio (TTS) |  
| | | \- User Uploads (ASR) |  
| | \+---------------------------+  
| |  
\+----------+-------------------------------------\>+---------------------------+  
| | Third-Party Services |  
| |---------------------------|  
\+-------------------------------------\>| \- Image Generation API |  
| \- TTS / ASR APIs |  
| \- Analytics Service |  
\+---------------------------+  
**Component Interaction Breakdown:**

1. **Client & Edge/CDN:** The user interacts with the Next.js client application, built with React and TypeScript. Vercel's Edge Network serves all static assets, cached server-rendered pages, and optimized images, ensuring low-latency delivery worldwide.3 This layer is the first point of contact and is critical for initial load performance.  
2. **API Layer:** User actions that require server interaction (e.g., fetching the next scene, submitting a choice) are handled by Next.js Route Handlers. These are serverless functions that execute synchronous business logic, query the database via Prisma, and return data to the client.2 This layer is responsible for the immediate, interactive aspects of the game.  
3. **Queue:** For long-running or resource-intensive tasks, the API layer does not wait for completion. Instead, it pushes a job description to a Redis queue (provided by services like Upstash or Vercel KV). This immediately returns a response to the client (e.g., "Image generation started"), decoupling the user experience from backend processing.  
4. **Background Workers:** A separate pool of serverless functions acts as background workers, listening for jobs on the Redis queue. These workers handle tasks like calling third-party AI APIs for image generation, processing audio for TTS or ASR, or running complex Spaced Repetition System (SRS) calculations for a user's entire vocabulary set. This ensures that heavy tasks do not block the main application threads and can be retried independently upon failure.  
5. **Database:** A PostgreSQL database, accessed via the Prisma ORM, serves as the single source of truth for all persistent data. This includes user accounts, the entire story graph (chapters, scenes, choices), player state (inventory, progress), and the detailed learning data (vocabulary, exposures, SRS state). The choice of a relational database is crucial for maintaining the complex relationships inherent in an RPG and learning system.6  
6. **Asset Storage & Third-Party Services:** All generated media assets (images, audio) are stored in an S3-compatible object storage service like Cloudflare R2 or AWS S3. These assets are served to the client through the CDN. The background workers are the primary interface with external APIs for AI-driven content generation and analysis.

This architecture provides the simplicity of a monorepo development environment while leveraging the power of a distributed, serverless backend. This model allows the application to scale specific functions independently—for example, scaling up image generation workers during peak usage—without affecting the core application's performance.8

### **1.2 Concrete Tech Stack & Rationale**

The selection of technologies is guided by principles of developer productivity, type safety, performance, and ecosystem maturity. The stack is centered around the Next.js and Vercel ecosystem, which provides a highly integrated and optimized foundation for building modern, full-stack applications.2

| Technology | Category | Rationale |
| :---- | :---- | :---- |
| **Next.js 14 (App Router)** | Framework | The server-centric model with React Server Components simplifies data fetching, reduces client-side JavaScript, and improves performance. The file-system-based routing is intuitive and powerful for organizing both pages and API endpoints.1 |
| **TypeScript** | Language | Essential for building a large, complex application. It provides compile-time type safety, which significantly reduces runtime errors, improves code maintainability, and enhances developer tooling with features like autocompletion.6 |
| **PostgreSQL** | Database | A robust, open-source relational database ideal for managing the complex, interconnected data of an RPG, including user state, inventory, and branching narrative structures. Its support for transactions and data integrity is critical.6 |
| **Prisma** | ORM | Provides a best-in-class, type-safe database client generated directly from the schema. This eliminates entire classes of data-related bugs and dramatically improves the developer experience when querying and mutating data.7 |
| **Tailwind CSS** | Styling | A utility-first CSS framework that enables rapid development of custom user interfaces without writing custom CSS. It promotes consistency and is highly maintainable, especially in large component-based applications.2 |
| **shadcn/ui** | UI Components | A collection of beautifully designed, accessible, and unstyled components built on top of Radix UI and Tailwind CSS. Components are copied into the project, giving full control over code and styling, avoiding dependency lock-in.6 |
| **Zustand & React Query** | Client State | Zustand is used for minimal, simple global client state (e.g., UI settings). React Query is the primary tool for managing server state, handling caching, re-fetching, and mutations of all data fetched from the API, simplifying data synchronization.2 |
| **Redis (Upstash / Vercel KV)** | In-Memory Store & Queue | A high-performance key-value store used for caching frequently accessed data (e.g., sessions) and as a message broker for the background job queue, enabling robust asynchronous processing. |
| **Howler.js / Web Audio API** | Audio | Howler.js provides a simple, reliable abstraction over the Web Audio API for handling all in-game audio, including background music, sound effects, and TTS narration, with broad browser compatibility. |
| **Framer Motion** | Animation | A production-ready motion library for React that makes it easy to create fluid, complex animations and micro-interactions, which are essential for an engaging, game-like user experience. |
| **Playwright & Jest** | Testing | Playwright is used for robust, end-to-end (E2E) testing of critical user flows in a real browser environment. Jest is used for unit and integration testing of individual components, utility functions, and business logic.5 |
| **Vercel** | Platform | The premier platform for deploying Next.js applications. It offers seamless Git integration, automatic deployments, serverless functions, a global edge network, and built-in optimizations for performance and scalability, significantly reducing DevOps overhead.3 |

## **II. The Data Core: Database Schema & State Management**

The database is the heart of the application, serving as the persistent source of truth for all content, player progress, and learning data. The schema is designed using the Prisma Schema Language (PSL) for clarity and type safety.10 It is architected to enforce relational integrity, optimize for common query patterns, and provide a flexible foundation for future feature development. A key design principle is the explicit separation of three distinct data domains:  
**Content** (the story itself), **Player State** (a user's journey and possessions), and **Learning State** (a user's pedagogical progress). This separation is fundamental to the system's maintainability, allowing content to be versioned independently of player data and enabling distinct analysis of gameplay versus learning progression.

### **2.1 Core Database Models**

The following table provides a high-level overview of the core database models and their primary relationships. This map serves as a guide to the more detailed Prisma schema that follows.

| Model Name | Description | Key Relationships |
| :---- | :---- | :---- |
| User | Stores player account information, authentication details, and preferences. | PlayerState, UserChoice, SrsState, VocabExposure |
| Story | The top-level container for a complete narrative adventure. | Chapter |
| Chapter | A major section of a Story, analogous to a book chapter or game level. | Story, Scene |
| Scene | A single node in the story graph, representing a location, dialogue exchange, or puzzle. | Chapter, Choice, Asset |
| Choice | A decision point offered to the player within a Scene. | Scene, ChoiceOutcome, UserChoice |
| ChoiceOutcome | Defines the consequences of a Choice, including state changes and branching. | Choice, Scene (next scene) |
| PlayerState | Tracks a user's current gameplay status, including location, stats (XP), and inventory. | User, InventoryItem |
| InventoryItem | Represents a specific item held by the player (e.g., clues, tools). | PlayerState, Asset |
| UserChoice | A log of every decision a player has made, creating a replayable path. | User, Choice |
| Achievement | Defines unlockable achievements and tracks which users have earned them. | User |
| VocabularyItem | A single Spanish word or phrase being taught, with its English translation. | VocabExposure, SrsState |
| SrsState | The current spaced repetition state for a specific user and vocabulary item. | User, VocabularyItem |
| VocabExposure | A log of every time a user interacts with a vocabulary item. | User, VocabularyItem |
| Asset | A generic model for media assets like images and audio files. | Scene, InventoryItem |
| ImageGenJob | Tracks the status of asynchronous image generation requests. | User (requester), Asset (result) |
| AnalyticsEvent | A flexible table for logging various user interaction events for analysis. | User |

### **2.2 Complete Prisma Schema**

The following schema.prisma file defines the complete database structure, including all fields, types, relations, and indexes. This schema serves as the blueprint for database migrations and the generated type-safe Prisma Client.7

Code snippet

// This is your Prisma schema file,  
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {  
  provider \= "prisma-client-js"  
}

datasource db {  
  provider \= "postgresql"  
  url      \= env("DATABASE\_URL")  
}

// \----------------------------------------  
// User and Authentication Models  
// \----------------------------------------  
model User {  
  id            String    @id @default(cuid())  
  email         String    @unique  
  name          String?  
  hashedPassword String?  
  createdAt     DateTime  @default(now())  
  updatedAt     DateTime  @updatedAt

  // Relations  
  playerState   PlayerState?  
  srsStates     SrsState  
  vocabExposures VocabExposure  
  userChoices   UserChoice  
  achievements  UserAchievement  
  analyticsEvents AnalyticsEvent  
  imageGenJobs  ImageGenJob  
}

// \----------------------------------------  
// Story Content Models  
// \----------------------------------------  
model Story {  
  id          String    @id @default(cuid())  
  title       String  
  description String?  
  authorId    String // Could link to an AdminUser model  
  published   Boolean   @default(false)  
  createdAt   DateTime  @default(now())  
  updatedAt   DateTime  @updatedAt

  // Relations  
  chapters    Chapter  
}

model Chapter {  
  id          String   @id @default(cuid())  
  storyId     String  
  title       String  
  chapterNumber Int  
  description String?  
  createdAt   DateTime @default(now())  
  updatedAt   DateTime @updatedAt

  // Relations  
  story       Story    @relation(fields: \[storyId\], references: \[id\], onDelete: Cascade)  
  scenes      Scene

  @@unique(\[storyId, chapterNumber\])  
}

model Scene {  
  id              String    @id @default(cuid())  
  chapterId       String  
  title           String // Internal title for the editor  
  sceneData       Json      // The full JSON content model for the scene  
  isStartingScene Boolean   @default(false)  
  createdAt       DateTime  @default(now())  
  updatedAt       DateTime  @updatedAt

  // Relations  
  chapter         Chapter   @relation(fields: \[chapterId\], references: \[id\], onDelete: Cascade)  
  choices         Choice  
  entryPointFor   ChoiceOutcome  
}

model Choice {  
  id        String   @id @default(cuid())  
  sceneId   String  
  text      String   // The text displayed to the player for this choice  
  order     Int      // Display order  
  createdAt DateTime @default(now())  
  updatedAt DateTime @updatedAt

  // Relations  
  scene     Scene           @relation(fields: \[sceneId\], references: \[id\], onDelete: Cascade)  
  outcomes  ChoiceOutcome  
  userChoices UserChoice  
}

model ChoiceOutcome {  
  id                String    @id @default(cuid())  
  choiceId          String  
  nextSceneId       String  
  description       String?   // Internal description of this outcome  
  conditions        Json?     // e.g., { "requiresItem": "key\_01", "statCheck": { "stat": "wisdom", "value": 5 } }  
  stateChanges      Json?     // e.g., { "addXP": 50, "setItem": "map\_piece\_02", "setFlag": "guard\_is\_angry" }  
  successProbability Float    @default(1.0)  
  createdAt         DateTime  @default(now())  
  updatedAt         DateTime  @updatedAt

  // Relations  
  choice            Choice    @relation(fields: \[choiceId\], references: \[id\], onDelete: Cascade)  
  nextScene         Scene     @relation(fields:, references: \[id\], onDelete: Restrict)  
}

// \----------------------------------------  
// Player State Models  
// \----------------------------------------  
model PlayerState {  
  id        String   @id @default(cuid())  
  userId    String   @unique  
  currentSceneId String?  
  xp        Int      @default(0)  
  storyFlags Json    @default("{}") // Stores persistent flags like {"guard\_is\_angry": true}  
  createdAt DateTime @default(now())  
  updatedAt DateTime @updatedAt

  // Relations  
  user      User     @relation(fields: \[userId\], references: \[id\], onDelete: Cascade)  
  inventory InventoryItem  
}

model InventoryItem {  
  id           String    @id @default(cuid())  
  playerStateId String  
  assetId      String  
  quantity     Int       @default(1)  
  acquiredAt   DateTime  @default(now())

  // Relations  
  playerState  PlayerState @relation(fields:, references: \[id\], onDelete: Cascade)  
  asset        Asset       @relation(fields: \[assetId\], references: \[id\])  
}

model UserChoice {  
  id         String   @id @default(cuid())  
  userId     String  
  choiceId   String  
  outcomeId  String   // The specific ChoiceOutcome that was triggered  
  timestamp  DateTime @default(now())

  // Relations  
  user       User     @relation(fields: \[userId\], references: \[id\], onDelete: Cascade)  
  choice     Choice   @relation(fields: \[choiceId\], references: \[id\], onDelete: Cascade)

  @@index(\[userId, timestamp\])  
}

model Achievement {  
  id          String   @id @default(cuid())  
  name        String   @unique  
  description String  
  iconUrl     String?  
  criteria    Json     // e.g., { "type": "CHOICE\_MADE", "choiceId": "xyz" }

  // Relations  
  userAchievements UserAchievement  
}

model UserAchievement {  
  userId        String  
  achievementId String  
  unlockedAt    DateTime @default(now())

  // Relations  
  user          User        @relation(fields: \[userId\], references: \[id\], onDelete: Cascade)  
  achievement   Achievement @relation(fields: \[achievementId\], references: \[id\], onDelete: Cascade)

  @@id(\[userId, achievementId\])  
}

// \----------------------------------------  
// Learning & SRS Models  
// \----------------------------------------  
model VocabularyItem {  
  id        String   @id @default(cuid())  
  spanish   String   @unique  
  english   String  
  audioAssetId String?  
  notes     String?  
  createdAt DateTime @default(now())

  // Relations  
  exposures VocabExposure  
  srsStates SrsState  
}

model VocabExposure {  
  id           String        @id @default(cuid())  
  userId       String  
  vocabId      String  
  exposureType ExposureType  // e.g., VIEWED, HEARD, SPOKE\_CORRECTLY  
  quality      Int           // Quality score (0-5) if applicable  
  timestamp    DateTime      @default(now())  
  sourceSceneId String?      // Where did this exposure happen?

  // Relations  
  user         User          @relation(fields: \[userId\], references: \[id\], onDelete: Cascade)  
  vocabularyItem VocabularyItem @relation(fields: \[vocabId\], references: \[id\], onDelete: Cascade)

  @@index(\[userId, timestamp\])  
}

enum ExposureType {  
  VIEWED\_TOOLTIP  
  HEARD\_NARRATION  
  USED\_IN\_CHOICE  
  SPOKE\_PRACTICE  
  REVIEW\_PASSED  
  REVIEW\_FAILED  
}

model SrsState {  
  id            String   @id @default(cuid())  
  userId        String  
  vocabId       String  
  repetitions   Int      @default(0)  
  intervalDays  Float    @default(0)  
  efactor       Float    @default(2.5)  
  nextReviewDate DateTime  
  lastReviewedAt DateTime?  
  updatedAt     DateTime @updatedAt

  // Relations  
  user          User         @relation(fields: \[userId\], references: \[id\], onDelete: Cascade)  
  vocabularyItem VocabularyItem @relation(fields: \[vocabId\], references: \[id\], onDelete: Cascade)

  @@unique(\[userId, vocabId\])  
  @@index()  
}

// \----------------------------------------  
// Asset & Job Models  
// \----------------------------------------  
enum AssetType {  
  IMAGE  
  AUDIO  
  VIDEO  
}

enum AssetStatus {  
  PENDING  
  APPROVED  
  REJECTED  
}

model Asset {  
  id        String      @id @default(cuid())  
  url       String  
  type      AssetType  
  name      String      // e.g., "spooky\_library.png"  
  altText   String?  
  metadata  Json?       // For AI gen: { prompt, seed, model, etc. }  
  status    AssetStatus @default(PENDING)  
  createdAt DateTime    @default(now())  
  updatedAt DateTime    @updatedAt

  // Relations  
  inventoryItems InventoryItem  
}

model ImageGenJob {  
  id        String   @id @default(cuid())  
  userId    String   // User who initiated the job (an admin)  
  prompt    String  
  status    String   // PENDING, PROCESSING, COMPLETED, FAILED  
  resultAssetId String?  @unique  
  error     String?  
  createdAt DateTime @default(now())  
  updatedAt DateTime @updatedAt

  // Relations  
  user      User     @relation(fields: \[userId\], references: \[id\])  
  asset     Asset?   @relation(fields: \[resultAssetId\], references: \[id\])  
}

// \----------------------------------------  
// Analytics Model  
// \----------------------------------------  
model AnalyticsEvent {  
  id        String   @id @default(cuid())  
  userId    String?  
  sessionId String  
  eventName String  
  payload   Json  
  timestamp DateTime @default(now())

  // Relations  
  user      User?    @relation(fields: \[userId\], references: \[id\], onDelete: SetNull)

  @@index(\[eventName, timestamp\])  
}

This schema is designed to be both comprehensive and extensible. The use of JSON fields for sceneData, conditions, and stateChanges provides flexibility for the content authoring team to define complex game logic without requiring database schema migrations for every new mechanic. Indexes are placed on foreign keys and fields frequently used in queries (e.g., userId and nextReviewDate on SrsState) to ensure high performance as the user base and content library grow.

## **III. Core Mechanics: Gameplay & Pedagogy**

The application's success hinges on the seamless fusion of two core systems: a pedagogically sound Spaced Repetition System (SRS) for language acquisition and an engaging RPG engine for narrative-driven motivation. These systems are not parallel tracks but are deeply intertwined, designed so that progress in one directly influences and enhances the experience in the other. This section details the data models, algorithms, and design patterns for these critical mechanics.

### **3.1 Spaced Repetition / Repetition Tracking Framework**

The SRS is the educational backbone of the application, designed to optimize long-term vocabulary retention. It is based on a variant of the well-established SM-2 algorithm, chosen for its simplicity, effectiveness, and extensive documentation.13 The framework is built around tracking various forms of "exposure" to vocabulary, ensuring that the learning schedule adapts to a rich set of user interactions, not just formal reviews.

#### **Data Models for SRS**

The learning process is captured in three core models: VocabularyItem, VocabExposure, and SrsState.

* **VocabularyItem**: Represents a single unit of learning (a word or phrase).  
* **VocabExposure**: A log entry created every time a user interacts with a VocabularyItem. This is a crucial data source, capturing the context and type of interaction. Exposure events include:  
  * **Passive Exposure:** VIEWED\_TOOLTIP (hovering over a word), HEARD\_NARRATION (hearing the word in dialogue).  
  * **Active Use:** USED\_IN\_CHOICE (selecting a dialogue option containing the word), SPOKE\_PRACTICE (attempting to pronounce the word).  
  * **Formal Review:** REVIEW\_PASSED, REVIEW\_FAILED (explicitly answering a review prompt correctly or incorrectly).  
* **SrsState**: This model holds the current state of the learning algorithm for each user-word pair. It contains the core SM-2 parameters: repetitions, intervalDays, and efactor (ease factor), which are updated after each meaningful review.

The Prisma schema for these models is as follows:

Code snippet

// From schema.prisma

model VocabularyItem {  
  id        String   @id @default(cuid())  
  spanish   String   @unique  
  english   String  
  audioAssetId String?  
  notes     String?  
  createdAt DateTime @default(now())

  // Relations  
  exposures VocabExposure  
  srsStates SrsState  
}

model VocabExposure {  
  id           String        @id @default(cuid())  
  userId       String  
  vocabId      String  
  exposureType ExposureType  
  quality      Int?          // Quality score (0-5) if applicable  
  timestamp    DateTime      @default(now())  
  sourceSceneId String?

  // Relations  
  user         User          @relation(fields: \[userId\], references: \[id\], onDelete: Cascade)  
  vocabularyItem VocabularyItem @relation(fields: \[vocabId\], references: \[id\], onDelete: Cascade)

  @@index(\[userId, timestamp\])  
}

enum ExposureType {  
  VIEWED\_TOOLTIP  
  HEARD\_NARRATION  
  USED\_IN\_CHOICE  
  SPOKE\_PRACTICE  
  REVIEW\_PASSED  
  REVIEW\_FAILED  
}

model SrsState {  
  id            String   @id @default(cuid())  
  userId        String  
  vocabId       String  
  repetitions   Int      @default(0)  
  intervalDays  Float    @default(0)  
  efactor       Float    @default(2.5) @db.Real  
  nextReviewDate DateTime  
  lastReviewedAt DateTime?  
  updatedAt     DateTime @updatedAt

  // Relations  
  user          User         @relation(fields: \[userId\], references: \[id\], onDelete: Cascade)  
  vocabularyItem VocabularyItem @relation(fields: \[vocabId\], references: \[id\], onDelete: Cascade)

  @@unique(\[userId, vocabId\])  
  @@index()  
}

#### **SRS Algorithm (SM-2 Variant)**

The core logic is encapsulated in a single function that takes the current SrsState and a quality score (an integer from 0 to 5, where 0 is a complete failure and 5 is a perfect recall) and returns the updated state.  
The TypeScript implementation of this function is provided below. It adheres strictly to the SM-2 logic for updating the ease factor and calculating the next review interval.13

TypeScript

// Located in /src/lib/srs.ts

type SrsState \= {  
  repetitions: number;  
  intervalDays: number;  
  efactor: number;  
};

type UpdatedSrsState \= SrsState & {  
  nextReviewDate: Date;  
};

/\*\*  
 \* Applies the SM-2 spaced repetition algorithm to update the SRS state.  
 \* @param srsState The current state of the item.  
 \* @param quality A number from 0-5 representing the quality of the recall.  
 \*                0: complete blackout  
 \*                1: incorrect response; correct one remembered  
 \*                2: incorrect response; correct one seemed easy to recall  
 \*                3: correct response recalled with serious difficulty  
 \*                4: correct response after a hesitation  
 \*                5: perfect response  
 \* @returns The updated SRS state including the next review date.  
 \*/  
export function applySm2(  
  srsState: SrsState,  
  quality: number  
): UpdatedSrsState {  
  if (quality \< 0 |

| quality \> 5\) {  
    throw new Error('Quality must be between 0 and 5.');  
  }

  let { repetitions, intervalDays, efactor } \= srsState;

  if (quality \>= 3\) {  
    // Correct response  
    if (repetitions \=== 0\) {  
      intervalDays \= 1;  
    } else if (repetitions \=== 1\) {  
      intervalDays \= 6;  
    } else {  
      intervalDays \= Math.ceil(intervalDays \* efactor);  
    }  
    repetitions \+= 1;  
  } else {  
    // Incorrect response  
    repetitions \= 0;  
    intervalDays \= 1;  
  }

  // Update ease factor (efactor)  
  efactor \= efactor \+ (0.1 \- (5 \- quality) \* (0.08 \+ (5 \- quality) \* 0.02));  
  if (efactor \< 1.3) {  
    efactor \= 1.3; // The ease factor should not be less than 1.3  
  }

  const now \= new Date();  
  const nextReviewDate \= new Date(  
    now.getFullYear(),  
    now.getMonth(),  
    now.getDate() \+ intervalDays  
  );

  return {  
    repetitions,  
    intervalDays,  
    efactor,  
    nextReviewDate,  
  };  
}

#### **Computation and Syncing Patterns**

To ensure a responsive user experience, the system employs a hybrid computation model and an optimistic update pattern for the client.

* **Computation:**  
  * **In-Request Updates:** When a user completes a single, high-value review (e.g., a formal review prompt), the applySm2 function is called directly within the API request handler. The SrsState for that single item is updated in the database before the response is sent. This is fast and provides immediate consistency.  
  * **Background Worker:** Less critical updates or batch processes (e.g., analyzing a day's worth of passive VocabExposure events to slightly adjust SRS states) are offloaded. A job is pushed to the Redis queue, and a background worker processes these updates asynchronously, preventing them from impacting API response times.  
* **Syncing:**  
  * The client uses an **optimistic update** strategy. When a user submits a review, the UI immediately reflects the outcome (e.g., moving the card to a "learned" state).  
  * Simultaneously, a request is sent to the server.  
  * If the server request succeeds, the optimistic state is confirmed.  
  * If the request fails (due to network issues, etc.), the UI reverts the change and displays an error message, ensuring the user interface remains consistent with the server's state of record.

#### **Surfacing Review Opportunities**

A critical design challenge is integrating SRS into a narrative game without breaking immersion.15 Abruptly showing a flashcard modal is jarring and antithetical to the "ClueFinders-style" experience. The system will instead surface review opportunities diegetically—that is, as a natural part of the game world.

1. **Review Pool:** The system maintains a "review pool" for each user, which is a list of VocabularyItems whose nextReviewDate is in the past.  
2. **Contextual Interleaving:** The story content is designed with natural breakpoints (e.g., the player character consults a journal, an NPC asks a question, a puzzle requires a password).  
3. **Diegetic Reviews:** At these breakpoints, the game logic queries the user's review pool. If items are available, they are woven into the gameplay:  
   * A locked door might require the player to type a review word as the password.  
   * A friendly character might ask, "¿Recuerdas cómo se dice 'key' en español?" ("Do you remember how to say 'key' in Spanish?"), presenting multiple-choice options drawn from the review pool.  
   * A clue found in the environment might be written in Spanish, using a word from the review pool.  
4. **Standalone Sessions:** For users who prefer a more traditional study method, an optional, non-narrative "Review Deck" will be accessible from the main menu, allowing them to clear their review queue in a more focused session.

This approach transforms SRS from a mandatory chore into a core game mechanic that helps the player solve puzzles and advance the story, creating a powerful, self-reinforcing loop of learning and engagement.

### **3.2 Branching Choice Tree & RPG Model**

The narrative of the game is not a linear path but a stateful, directed graph. This structure allows for meaningful player agency, where choices have persistent consequences that shape the unfolding story.17 This is achieved through a combination of the  
Scene, Choice, and ChoiceOutcome database models, which together create a powerful and flexible RPG system.

#### **Schema for Branching Narratives**

* **Scene**: A node in the graph. It contains the content for a specific moment in the story (dialogue, environment description, puzzle).  
* **Choice**: An edge leading from a Scene. It represents a decision the player can make.  
* **ChoiceOutcome**: The logic that defines the result of a Choice. This is the core of the RPG system. It specifies the nextSceneId the player will transition to, but more importantly, it can define conditions for the choice to be available and state changes that occur when the choice is made.

This model moves beyond a simple branching tree, where a choice only leads to a new branch. By incorporating conditions and state changes, it creates a reactive world where the availability of future choices can depend on the entire history of a player's decisions.19

#### **Mechanics for Risk/Reward**

The ChoiceOutcome model's conditions and stateChanges JSON fields enable a wide variety of "ClueFinders-style" risk/reward mechanics, making choices feel impactful.

* **Stat Checks:** A choice can be made available only if the player meets a certain condition stored in their PlayerState. For example, a Choice to persuade a guard might have a ChoiceOutcome with a conditions block like: {"statCheck": {"stat": "charisma", "value": 10}}. The game logic would check if the player's charisma stat is greater than or equal to 10 before showing this option.  
* **Inventory Usage:** A choice can require a specific InventoryItem. The conditions could be {"requiresItem": "ancient\_key"}. If the player makes this choice, the stateChanges block would specify {"consumeItem": "ancient\_key"}, removing the item from their inventory.  
* **XP and Progression:** ChoiceOutcomes are the primary way to reward players. A stateChanges block can include {"addXP": 50} to grant experience points or {"setFlag": "has\_map\_piece\_3"} to track progress on a larger quest.  
* **Time-Limited Choices:** The Scene data model can include a timeLimitSeconds property for specific scenes. The client-side SceneRenderer will display a countdown timer, and if the timer expires, the system can trigger a default ChoiceOutcome, adding a sense of urgency.  
* **Probability Outcomes:** A single Choice can have multiple ChoiceOutcomes, each with a successProbability between 0 and 1\. When the player makes the choice, the server rolls a die to determine which outcome occurs. This can be used for actions like trying to pick a lock, where there is a chance of both success and failure, adding replayability and unpredictability.

#### **Recording and Analyzing Player Paths**

Every decision a player makes is recorded in the UserChoice table. This table stores the userId, the choiceId they selected, and the specific outcomeId that was triggered. This creates an immutable, chronological log of each player's unique journey through the story graph.  
This data is immensely valuable for several reasons:

1. **Replay and Debugging:** Developers can reconstruct a player's exact path to debug issues or understand unexpected game states.  
2. **Player Analytics:** Product managers and game designers can analyze the aggregated data to understand player behavior. Which story branches are the most popular? Where do most players get stuck? Which choices lead to players abandoning the game? This analysis can inform future content development and balancing.21  
3. **Learning Analytics:** By joining UserChoice data with VocabExposure data, instructional designers can investigate correlations between narrative choices and learning outcomes. For example, do players who choose more inquisitive dialogue options demonstrate faster vocabulary acquisition? This allows for data-driven improvements to the game's pedagogical design.

### **3.3 Exploration, Clues, and Object-Collection Mechanics**

A core tenet of the "ClueFinders-style" experience is interactive exploration and puzzle-solving that is deeply integrated with the learning objectives.22 Players should feel like detectives, piecing together clues from their environment to overcome obstacles, with language learning being the key to their success.

#### **Modeling Interactive Environments**

To facilitate exploration, each Scene's sceneData JSON blob can contain an array of interactiveHotspots. Each hotspot object defines a clickable area within the scene's background image and the consequences of interacting with it.  
An example interactiveHotspots array:

JSON

"interactiveHotspots":

This data-driven approach allows content creators to design rich, interactive scenes without writing any code. The client-side SceneRenderer simply iterates over this array, makes the areas clickable, and executes the defined actions upon user interaction.

#### **Example Puzzle Design: Contextual Language Clues**

This system enables the creation of puzzles where language knowledge is not just tested but is the tool for solving the problem itself.24  
**Scenario: The Locked Chest**

1. **Objective:** The player must open a locked chest to retrieve a critical story item.  
2. **Exploration & Clue Discovery:** In the scene, the player explores the environment by clicking on hotspots. They click on the painting (hotspot\_painting\_01) and read the plaque: *"La palabra secreta es la primera luz del día."* (The secret word is the first light of day). This interaction logs a VIEWED\_TOOLTIP exposure for the vocabulary items in the sentence.  
3. **Puzzle Interaction:** The player then clicks on the locked chest (hotspot\_chest\_01). This triggers a PUZZLE\_INPUT action, presenting them with a text input field.  
4. **Solution & Reinforcement:** The player must deduce the Spanish word for "dawn," which is amanecer. They type amanecer into the input field.  
5. **Backend Verification:** The client sends the solution to the server. The server verifies that amanecer is the correct answer for puzzle\_dawn\_word.  
6. **Reward & Learning Feedback:** Upon successful verification, the server triggers a stateChange (e.g., adds the chest's contents to the player's inventory) and, crucially, logs a high-quality REVIEW\_PASSED exposure event for the VocabularyItem 'amanecer'. The player has not only recalled a word but has *used it contextually to achieve a goal*, which is a far more powerful learning experience than a simple flashcard review. This direct link between language skill and game progression is the essence of the design philosophy.

## **IV. Content & Asset Generation Pipelines**

To create a visually rich and audibly immersive world at scale, the application will rely on automated pipelines for generating image and audio assets. These pipelines are designed to be efficient, consistent, and maintain a high standard of quality through a combination of AI-powered generation and human-in-the-loop oversight.

### **4.1 Image Generation Pipeline**

The image generation pipeline is responsible for creating unique background art, character portraits, and item icons that adhere to a consistent, "ClueFinders-style" aesthetic. The process is managed through a job queue to handle asynchronous generation and includes a critical manual approval step for quality control.

#### **Prompt Templating and Style Consistency**

Maintaining a consistent art style across thousands of generated images is a significant challenge. The solution is a rigorous prompt templating strategy.25 Instead of allowing free-form prompts, the system will construct prompts from a structured template.  
A base "style prompt" will be defined in the application's configuration to encapsulate the core aesthetic. For example:  
"masterpiece, best quality, 90s educational adventure game art, vibrant colors, clean outlines, cartoon style, no text, no artifacts"  
When an author requests an image, they provide only the subject-specific details. The system then combines this with the style prompt to create the final prompt sent to the AI provider.  
**Metadata Storage:** For every generated asset, the Asset table's metadata field will store critical information for reproducibility and debugging:

* fullPrompt: The exact text sent to the API.  
* negativePrompt: Any negative prompts used.  
* seed: The generation seed, crucial for recreating an image.  
* model: The specific AI model and version used (e.g., stable-diffusion-xl-v1.0).  
* styleTemplateVersion: The version of the base style prompt used.  
* generationParams: Other parameters like cfg\_scale, steps, sampler.

#### **Asynchronous Job-Based Workflow**

The generation process is asynchronous to prevent blocking the admin UI.

1. **Job Creation:** An author defines an image request in the admin panel (e.g., "A mysterious, ancient library filled with glowing books"). This creates a new entry in the ImageGenJob table with a PENDING status and pushes a job message to the Redis queue.  
2. **Worker Processing:** A background worker picks up the job. It constructs the full prompt, makes the API call to the image generation provider (e.g., Stability AI, Midjourney API), and updates the job status to PROCESSING.  
3. **Retry Policy:** If the API call fails or times out, the worker will implement an exponential backoff retry policy (e.g., wait 2s, then 4s, then 8s) to handle transient network issues. After a set number of failures, the job is marked as FAILED and an alert is sent to the administrators.  
4. **Asset Storage and Caching:** Upon successful generation, the worker uploads the image file to the S3-compatible object storage. It then creates a new Asset record in the database with the image URL and all relevant metadata. The ImageGenJob is updated to COMPLETED and linked to the new Asset.  
5. **CDN Delivery:** All approved assets are served to the end-user via the Vercel CDN, which provides global caching and on-the-fly image optimization (e.g., converting to modern formats like WebP, resizing) to ensure fast delivery.3

#### **Content Moderation and Admin Approval**

A fully automated pipeline is a risk to brand safety and quality. Therefore, a human-in-the-loop approval process is a non-negotiable part of the workflow.

1. **Approval Queue:** Newly generated assets are created with a status of PENDING. The admin panel features an "Approval Queue" screen that displays all pending images.  
2. **Manual Review:** A content moderator reviews each image for quality, style consistency, and appropriateness. They can choose to APPROVE or REJECT the asset.  
3. **Fallback Assets:** If an AI-generated image is repeatedly rejected or if a very specific image is required, the system allows an author to manually upload a "fallback" asset and associate it with a scene. This ensures that content production is never blocked by the limitations of the AI generator. An approved, manually uploaded asset will always take precedence over a pending or rejected AI-generated one.

This pipeline balances the speed and scalability of AI generation with the quality control and safety of human oversight, ensuring a high-quality and consistent visual experience for the player.

### **4.2 Interactive Audio Pipeline**

The audio pipeline provides both the narrative voice of the game (Text-to-Speech) and a mechanism for players to practice their pronunciation (Automatic Speech Recognition).

#### **Text-to-Speech (TTS) Design**

For static content like story narration and character dialogue, a **pre-generation** strategy is the most performant and cost-effective approach.

* **Process:** When a story chapter is finalized and approved in the admin panel, a background job is triggered. This job iterates through every line of dialogue in the chapter, sends the text to a high-quality neural TTS provider (e.g., ElevenLabs, Google Cloud TTS, Azure Speech Service), and saves the resulting audio file (e.g., MP3 or Opus) to the S3 bucket.27  
* **Storage and Streaming:** Each audio file is stored with a predictable name (e.g., scene\_{sceneId}\_line\_{lineId}.mp3) and an Asset record is created in the database linking the audio URL to the specific line of dialogue. On the client, audio is streamed from the CDN, not downloaded all at once, to ensure fast scene loading times.  
* **Voice Selection:** The system will support multiple voices to differentiate between the narrator and various characters. The specific voice to be used for a line of dialogue will be part of the scene's content model, allowing authors to "cast" voices for their characters.

#### **Automatic Speech Recognition (ASR) and Pronunciation Scoring**

To provide players with meaningful feedback on their spoken Spanish, the system will use a two-tiered ASR approach.

1. **Client-Side (Lightweight Feedback):** For informal, in-context practice (e.g., a button next to a vocabulary word that says "Try saying it\!"), the application will use the browser's built-in **Web Speech API**. This provides instant, real-time transcription. While it lacks detailed scoring, it's free and sufficient for low-stakes, immediate feedback.  
2. **Server-Side (Detailed Scoring):** For formal challenges and assessments that impact the SRS, a more robust server-side approach is used.  
   * **Recording:** The client records the user's speech and sends the audio blob to a dedicated API endpoint.  
   * **Specialized ASR:** The backend forwards this audio to a specialized ASR service designed for language learning, such as **Speechace**.30 Unlike general-purpose transcription services, these APIs provide rich, structured feedback.  
   * **Pronunciation Feedback:** The API response will include not just a transcript, but a detailed analysis of the pronunciation, including:  
     * An overall accuracy score (e.g., 85%).  
     * Word-level confidence scores.  
     * **Phoneme-level analysis**, highlighting specific sounds that were mispronounced (e.g., "The 'rr' in 'perro' was not rolled correctly").31  
     * Fluency metrics like speaking rate and pausing.  
   * **Feedback Loop:** This detailed feedback is presented to the user in a clear, actionable format. The overall accuracy score is then normalized to a 0-5 scale and used as the quality input for the applySm2 SRS function. A high pronunciation score results in a longer review interval, directly rewarding the player's speaking practice.

All audio assets, including user-submitted recordings for ASR, are stored in the S3 bucket. A clear privacy policy and data retention schedule must be in place for user-generated audio to comply with regulations like GDPR and CCPA.

## **V. Implementation & User Experience**

This section translates the high-level architecture and mechanics into an actionable implementation plan for the development team. It covers the project's file structure, the data contract for content, API endpoint definitions, the design of the crucial admin authoring tool, the client-side component architecture, and the strategy for enabling offline gameplay.

### **5.1 Project File & Folder Structure**

A well-organized project structure is essential for maintainability and scalability, especially in a full-stack Next.js application.35 The proposed structure leverages the conventions of the Next.js App Router, including the use of a  
src directory and route groups to cleanly separate application concerns.9

/my-spanish-rpg  
├── public/                     \# Static assets (images, fonts, etc.)  
├── src/  
│   ├── app/                    \# Next.js App Router  
│   │   ├── (game)/             \# Player-facing game application  
│   │   │   ├── layout.tsx  
│   │   │   └── story/\[storyId\]/\[chapterId\]/page.tsx  
│   │   ├── (admin)/            \# Internal admin & authoring tool  
│   │   │   ├── layout.tsx  
│   │   │   └── story-editor/\[storyId\]/page.tsx  
│   │   ├── api/                \# API Route Handlers  
│   │   │   ├── story/progress/route.ts  
│   │   │   ├── learning/exposure/route.ts  
│   │   │   └── jobs/image-gen/route.ts  
│   │   ├── layout.tsx          \# Root application layout  
│   │   └── page.tsx            \# Landing page  
│   ├── components/             \# Reusable React components  
│   │   ├── game/               \# Components specific to the game UI  
│   │   │   ├── SceneRenderer.tsx  
│   │   │   └── DialogueBox.tsx  
│   │   ├── admin/              \# Components specific to the admin panel  
│   │   │   └── StoryNodeEditor.tsx  
│   │   └── ui/                 \# Generic UI primitives (from shadcn/ui)  
│   │       ├── Button.tsx  
│   │       └── Tooltip.tsx  
│   ├── lib/                    \# Shared business logic, utilities, and services  
│   │   ├── srs.ts              \# Spaced Repetition algorithm implementation  
│   │   ├── db.ts               \# Prisma client singleton instance  
│   │   ├── jobs.ts             \# Functions for creating background jobs  
│   │   ├── auth.ts             \# Authentication configuration and helpers  
│   │   └── utils.ts            \# General utility functions  
│   ├── styles/                 \# Global CSS styles  
│   │   └── globals.css  
│   └── types/                  \# Shared TypeScript type definitions  
│       └── index.ts  
├── prisma/  
│   └── schema.prisma           \# Database schema definition  
├──.env.local                  \# Environment variables  
├── next.config.mjs             \# Next.js configuration  
└── tsconfig.json               \# TypeScript configuration

The use of route groups—(game) and (admin)—is a key organizational pattern. These folders group routes for organizational purposes without affecting the URL structure. This allows for separate root layouts, authentication middleware, and component sets for the player-facing game and the internal admin tool, maintaining a clean separation of concerns within a single Next.js application.9

### **5.2 Example Content Model for a Single Scene**

The SceneRenderer component on the client will be driven by a JSON data structure fetched from the server. This object, stored in the sceneData field of the Scene model, acts as the contract between the backend and frontend. It contains all the information needed to render a complete, interactive scene.  
Below is an example of a JSON object for a single scene, demonstrating how various elements like bilingual text, SRS tags, audio narration, and interactive choices are represented.

JSON

{  
  "sceneId": "scene\_ch1\_001",  
  "title": "The Dusty Office",  
  "backgroundAssetId": "asset\_dusty\_office\_bg",  
  "musicAssetId": "asset\_music\_mystery\_loop",  
  "dialogue":,  
      "translation": "You find yourself in a dusty office. A single key sits on the desk."  
    },  
    {  
      "speaker": "Misterio",  
      "lineId": "line\_002",  
      "audioAssetId": "audio\_ch1\_001\_002",  
      "text": \[  
        { "type": "text", "content": "¿Qué vas a hacer? El " },  
        { "type": "vocab", "content": "tiempo", "vocabId": "vocab\_time" },  
        { "type": "text", "content": " es corto." }  
      \],  
      "translation": "What are you going to do? Time is short."  
    }  
  \],  
  "choices":,  
  "interactiveHotspots":  
}

This structure is highly flexible. The dialogue.text array allows for mixing plain text with interactive vocabulary words, which the frontend can render as tappable elements with tooltips.

### **5.3 API Endpoints (Next.js Route Handlers)**

The application's API will be built using Next.js Route Handlers, which provide a clean and conventional way to create RESTful endpoints within the app/api directory. The following table outlines the core endpoints, defining the API contract for frontend and backend developers.

| Endpoint | HTTP Method | Description | Request Body (Example) | Response Body (Example) |
| :---- | :---- | :---- | :---- | :---- |
| /api/story/scene/{sceneId} | GET | Fetches the complete data model for a single scene. | \- | { "sceneData": {... } } |
| /api/story/choice | POST | Submits a player's choice and returns the outcome. | { "choiceId": "...", "sceneId": "..." } | { "outcome": {... }, "nextSceneId": "..." } |
| /api/learning/exposure | POST | Records a vocabulary exposure event (e.g., tooltip view). | { "vocabId": "...", "exposureType": "VIEWED\_TOOLTIP" } | { "status": "ok" } |
| /api/learning/review | POST | Submits a formal SRS review and returns the updated SRS state. | { "vocabId": "...", "quality": 4 } | { "updatedSrsState": {... } } |
| /api/learning/asr | POST | Submits a user's audio recording for pronunciation scoring. | FormData with audioBlob and vocabId | { "score": 92, "feedback": {... } } |
| /api/jobs/image-gen | POST | (Admin) Creates a new asynchronous image generation job. | { "prompt": "A spooky castle" } | { "jobId": "...", "status": "PENDING" } |
| /api/jobs/image-gen/{jobId} | GET | (Admin) Checks the status of an image generation job. | \- | { "jobId": "...", "status": "COMPLETED", "assetUrl": "..." } |
| /api/admin/story | POST | (Admin) Creates a new story. | { "title": "..." } | { "story": {... } } |

All endpoints will be protected by authentication middleware to ensure that only authorized users can access or modify data.

### **5.4 Admin & Authoring UX**

A powerful and intuitive admin panel is critical for enabling non-technical writers, instructional designers, and content managers to create and manage the game's narrative content. The admin UX will be a first-class part of the application, not an afterthought.38  
Key screens and features will include:

* **Story Editor:** The centerpiece of the authoring experience will be a visual, node-based editor. Each Scene will be represented as a card, and Choices will be represented as lines connecting the cards. This provides a clear, high-level overview of the story's branching structure, similar to tools like Twine or Articy Draft.40 Authors can drag and drop to rearrange scenes and visually map out complex narrative flows.  
* **Scene Editor:** Clicking on a scene node will open a detailed editor form. This form will allow authors to write dialogue, tag vocabulary words, upload or request background images, assign audio files, and define interactive hotspots and puzzles.  
* **Image Prompt Editor & Approval Queue:** A dedicated interface for managing the image generation pipeline. Authors can write and test prompts, view generated images, and then approve or reject them. This human-in-the-loop system is essential for maintaining visual quality and brand safety.  
* **Media Library:** A central repository for all approved images, audio files, and other assets, allowing for easy reuse across different scenes and stories.  
* **SRS Tagger Tools:** Tools for easily tagging vocabulary within dialogue and bulk-managing the VocabularyItem database.  
* **Playtest Preview:** Authors must be able to instantly preview and play through the story as they build it, allowing for rapid iteration and testing of narrative flow and puzzle mechanics.  
* **Analytics Dashboards:** Visual dashboards displaying key metrics, such as player progression funnels, most common choice paths, and vocabulary item mastery rates, to inform content improvements.

### **5.5 Client Architecture & Component Breakdown**

The client-side application will be built with React, leveraging the modern features of the Next.js App Router.

* **State Management:** The architecture will heavily favor Server Components for fetching and rendering initial data, minimizing the amount of client-side JavaScript.1 For client-side state, a combination of  
  **Zustand** and **React Query** will be used. Zustand will manage minimal global UI state (e.g., audio volume settings), while React Query will handle all server state: caching API responses, managing loading and error states for data fetching, and orchestrating data mutations (e.g., submitting a choice).  
* **Component Breakdown:** The UI will be broken down into a series of logical, reusable components:  
  * SceneRenderer: The main parent component that orchestrates the entire game view. It receives the scene data and renders the appropriate sub-components.  
  * ImageLayer: Renders the background image and interactive hotspots.  
  * DialogueBox: Displays the current line of dialogue, speaker name, and provides controls for advancing the text.  
  * ChoiceList: Renders the list of available choices for the player.  
  * AudioPlayer: A global component, managed by Zustand, that handles background music and narration playback.  
  * VocabularyTooltip: A component that appears when a user hovers over or taps a tagged vocabulary word, showing its translation and a play-audio button.  
  * InventoryPanel: A slide-out panel that displays the items the player has collected.  
  * ReviewModal: A modal used for the optional, standalone SRS review sessions.  
* **Animations and Accessibility:**  
  * **Animations:** Framer Motion will be used to add fluid transitions between scenes, animate UI elements, and provide satisfying feedback for user interactions, enhancing the game-like feel.  
  * **Accessibility (A11y):** The application will be designed with accessibility in mind. This includes full keyboard navigation for all interactive elements, proper use of ARIA attributes to provide context for screen readers, and providing text captions or transcripts for all audio content.

### **5.6 Offline-First & Caching Strategy for Mobile (PWA)**

To provide a reliable experience on mobile devices with intermittent connectivity, the application will be a Progressive Web App (PWA) with a robust offline-first strategy.42

* **Service Worker:** A service worker will be implemented (using a library like serwist or a custom script) to intercept network requests and manage caching.44  
* **Caching Strategy:**  
  * **App Shell:** On first visit, the service worker will cache the core application shell—the essential JavaScript, CSS, and font files needed to run the app.  
  * **Chapter-Aware Prefetching:** Caching the entire game's assets upfront is impractical. Instead, when a user starts a new chapter online, the service worker will be triggered to proactively download and cache all required assets for that chapter (all scene data, background images, and audio files). This ensures a seamless offline experience for the current section of the game.  
* **Offline Data Synchronization:**  
  * **Local Storage:** Player progress made while offline (e.g., UserChoice records, VocabExposure events) will be stored locally in **IndexedDB**, which is well-suited for storing structured data.  
  * **Sync Queue:** Each offline action will be added to a sync queue in IndexedDB.  
  * **Background Sync:** When the application detects that network connectivity has been restored, a background sync process will be initiated. It will send the queued actions to the server in the order they were performed.  
  * **Conflict Resolution:** For this application's data model, a "last write wins" strategy is generally sufficient. The server will process the queued updates from the client. Since player state is largely append-only (new choices, new exposures), complex conflict resolution is minimized.46 The UI will provide clear indicators of the current sync status (e.g., "Offline," "Syncing," "All changes saved") to keep the user informed.48

## **VI. Strategic & Operational Framework**

This final section outlines the strategic and operational plan for bringing the application from concept to a robust, production-ready service. It covers the product roadmap, a comprehensive testing strategy, critical security and privacy considerations, and a plan for ensuring performance and scalability.

### **6.1 Roadmap & MVP**

A phased approach to development is recommended to validate the core concept, gather user feedback, and manage complexity. The roadmap is broken into three major milestones: Minimum Viable Product (MVP), Version 1 (V1), and Version 2 (V2).

* **Milestone 1: Minimum Viable Product (MVP)**  
  * **Goal:** Validate the core game loop of narrative-driven language learning.  
  * **Acceptance Criteria:**  
    * One complete, playable story chapter with a clear beginning, middle, and end.  
    * Core branching narrative engine is functional: players can make choices that lead to different scenes.  
    * Basic SRS is implemented: VocabularyItems encountered in the chapter are added to the user's SrsState, and the applySm2 algorithm schedules reviews.  
    * A simple, standalone review interface is available.  
    * All images and audio are manually created and uploaded by the admin; no AI generation pipelines are included.  
    * Basic user authentication (signup, login) is in place.  
* **Milestone 2: Version 1 (V1)**  
  * **Goal:** Enhance the content production pipeline and deepen the learning mechanics.  
  * **Acceptance Criteria:**  
    * The full image generation pipeline is operational, including the prompt editor and admin approval queue.  
    * The Text-to-Speech (TTS) pipeline for pre-generating narration audio is complete.  
    * The server-side Automatic Speech Recognition (ASR) pipeline is integrated, providing detailed pronunciation scoring and feedback.  
    * The content library is expanded to include multiple story chapters.  
    * Diegetic review opportunities are integrated into the narrative flow.  
* **Milestone 3: Version 2 (V2)**  
  * **Goal:** Broaden the RPG elements and introduce advanced features for engagement and accessibility.  
  * **Acceptance Criteria:**  
    * Advanced RPG mechanics are implemented, such as player stats (charisma, wisdom), stat checks on choices, and more complex inventory-based puzzles.  
    * The Progressive Web App (PWA) is fully implemented with the chapter-aware offline-first strategy.  
    * Social features, such as leaderboards for XP or vocabulary mastery, are introduced.  
    * Analytics-driven personalization is explored, potentially adapting story difficulty or vocabulary density based on player performance.

### **6.2 Testing & QA Plan**

A multi-layered testing strategy is essential to ensure application quality, covering everything from individual functions to complete user journeys.5

* **Unit Testing (Jest):**  
  * **Target:** Pure functions with no side effects.  
  * **Examples:** The applySm2 SRS algorithm, utility functions for formatting text, validation logic for API inputs.  
  * **Goal:** Verify the correctness of core business logic in isolation.  
* **Integration Testing (Jest & React Testing Library):**  
  * **Target:** Interactions between components, and components with mocked APIs.  
  * **Examples:** Testing that clicking a choice button correctly triggers a mutation hook; verifying that the SceneRenderer correctly displays data passed to it; testing Prisma queries against a dedicated test database to ensure they return the expected data structures.  
  * **Goal:** Ensure that different parts of the application work together as intended.  
* **End-to-End (E2E) Testing (Playwright):**  
  * **Target:** Critical user flows simulated in a real browser environment.  
  * **Scenarios:**  
    1. **Golden Path:** A user signs up, starts the first chapter, makes a series of choices to reach the end, and logs out.  
    2. **Branching Path Verification:** A test that makes a specific choice and asserts that the correct conditional scene is loaded next.  
    3. **SRS Loop:** A user encounters a new word, completes a review for it, and the test verifies that the nextReviewDate is correctly updated in the database.  
    4. **Image Generation Flow:** An admin logs in, requests a new image, waits for the job to complete (by polling the status endpoint), and verifies the image appears in the approval queue.  
  * **Goal:** Validate that the entire integrated system functions correctly from the user's perspective.  
* **Manual QA:**  
  * Manual testing will be crucial for aspects that are difficult to automate, such as the subjective quality of the narrative, the fun factor of puzzles, the aesthetic consistency of generated images, and the naturalness of TTS audio.

### **6.3 Security, Privacy & Compliance**

Building a secure and compliant application is a foundational requirement, particularly when handling user data and voice recordings.

* **Authentication & Authorization:** Use a battle-tested authentication solution like NextAuth.js or Clerk. Implement role-based access control (RBAC) to ensure that only administrators can access the admin panel and authoring APIs.  
* **API Security:**  
  * **Rate Limiting:** Apply strict rate limits to computationally expensive and costly API endpoints, especially /api/jobs/image-gen and /api/learning/asr, to prevent abuse and control costs.  
  * **Input Validation:** Use a library like Zod to validate all incoming API request bodies to prevent injection attacks and ensure data integrity.9  
  * **Data Security:** All data will be encrypted in transit (TLS/SSL) and at rest (standard for modern database providers). Sensitive user information should be minimized and handled with care.  
* **Privacy & Compliance (GDPR/CCPA):**  
  * **Voice Consent:** Before a user can use the ASR pronunciation feature, they must be presented with a clear and explicit consent form explaining that their voice will be recorded and sent to a third-party service for analysis.  
  * **Data Retention Policy:** Establish and publish a clear data retention policy. User-generated audio recordings should be stored only as long as necessary for analysis and should be anonymized or deleted upon user request or after a defined period.  
  * **PII:** Minimize the collection of Personally Identifiable Information (PII). Ensure all user data can be exported or deleted upon request to comply with "right to access" and "right to be forgotten" regulations.

### **6.4 Performance & Scaling**

The serverless architecture is inherently scalable, but performance must be considered at every layer of the stack.

* **Database:**  
  * **Indexing:** The Prisma schema includes indexes on columns that are frequently used in WHERE clauses (e.g., userId, nextReviewDate, eventName). These are critical for maintaining fast query performance as tables grow.  
  * **Connection Pooling:** Use a connection pooler like Prisma Accelerate or PgBouncer to efficiently manage database connections in a serverless environment, preventing connection exhaustion under load.  
* **CDN & Caching:**  
  * Aggressively leverage Vercel's Edge CDN to cache all static assets, images, audio, and the output of Next.js Server Components.  
  * Use appropriate Cache-Control headers, including stale-while-revalidate, to serve cached content quickly while revalidating it in the background.3  
* **Backend Services:**  
  * **Worker Autoscaling:** Configure the background worker infrastructure (e.g., Vercel Functions or AWS Lambda) to automatically scale the number of concurrent executions based on the length of the Redis job queue. This ensures that bursts in generation requests are handled efficiently without manual intervention.  
  * **Redis:** Use Redis not only for the job queue but also for caching expensive database queries or frequently accessed, non-critical data to reduce load on the primary PostgreSQL database.  
* **Client-Side Performance:**  
  * **Code Splitting:** Next.js automatically splits code by route. Further optimize by dynamically importing large components or libraries that are not needed on the initial page load.  
  * **Lazy Loading:** Lazy load assets, especially images and audio for scenes that are not immediately visible to the user, to improve initial load times.  
  * **Bundle Analysis:** Regularly use tools like the @next/bundle-analyzer to inspect the client-side JavaScript bundle and identify opportunities for optimization.

#### **Works cited**

1. Next.js Docs: App Router, accessed October 4, 2025, [https://nextjs.org/docs/app](https://nextjs.org/docs/app)  
2. A Software Architect's Guide to Choosing Next.js for Your Project \- Adhithi Ravichandran, accessed October 4, 2025, [https://adhithiravi.medium.com/a-software-architects-guide-to-choosing-next-js-for-your-project-f6a541133c41](https://adhithiravi.medium.com/a-software-architects-guide-to-choosing-next-js-for-your-project-f6a541133c41)  
3. Next.js on Vercel, accessed October 4, 2025, [https://vercel.com/docs/frameworks/full-stack/nextjs](https://vercel.com/docs/frameworks/full-stack/nextjs)  
4. Pages Router: Deploy to Vercel \- Next.js, accessed October 4, 2025, [https://nextjs.org/learn/pages-router/deploying-nextjs-app-deploy](https://nextjs.org/learn/pages-router/deploying-nextjs-app-deploy)  
5. App Router: Building Your Application \- Next.js, accessed October 4, 2025, [https://nextjs.org/docs/14/app/building-your-application](https://nextjs.org/docs/14/app/building-your-application)  
6. Seeking Advice on the Best Tech Stack : r/nextjs \- Reddit, accessed October 4, 2025, [https://www.reddit.com/r/nextjs/comments/1irsbwd/seeking\_advice\_on\_the\_best\_tech\_stack/](https://www.reddit.com/r/nextjs/comments/1irsbwd/seeking_advice_on_the_best_tech_stack/)  
7. Mastering Database Interactions with Prisma ORM: A Modern Developer's Toolkit \- JIITAK, accessed October 4, 2025, [https://www.jiitak.com/blog/mastering-database-interactions-with-prisma-orm-a-modern-developers-toolkit](https://www.jiitak.com/blog/mastering-database-interactions-with-prisma-orm-a-modern-developers-toolkit)  
8. How To Choose The Right Technology Stack For eLearning App Development?, accessed October 4, 2025, [https://www.zealousys.com/blog/technology-stack-for-elearning-app-development/](https://www.zealousys.com/blog/technology-stack-for-elearning-app-development/)  
9. Next.js App Router: Project Structure \- Makerkit, accessed October 4, 2025, [https://makerkit.dev/blog/tutorials/nextjs-app-router-project-structure](https://makerkit.dev/blog/tutorials/nextjs-app-router-project-structure)  
10. Prisma Schema Language: The Best Way to Define Your Data, accessed October 4, 2025, [https://www.prisma.io/blog/prisma-schema-language-the-best-way-to-define-your-data](https://www.prisma.io/blog/prisma-schema-language-the-best-way-to-define-your-data)  
11. Prisma | Instant Postgres plus an ORM for simpler db workflows, accessed October 4, 2025, [https://www.prisma.io/](https://www.prisma.io/)  
12. Getting Started: Deploying \- Next.js, accessed October 4, 2025, [https://nextjs.org/docs/app/getting-started/deploying](https://nextjs.org/docs/app/getting-started/deploying)  
13. thyagoluciano/sm2: SM-2 is a simple spaced repetition algorithm. It calculates the number of days to wait before reviewing a piece of information based on how easily the the information was remembered today. \- GitHub, accessed October 4, 2025, [https://github.com/thyagoluciano/sm2](https://github.com/thyagoluciano/sm2)  
14. What spaced repetition algorithm does Anki use?, accessed October 4, 2025, [https://faqs.ankiweb.net/what-spaced-repetition-algorithm](https://faqs.ankiweb.net/what-spaced-repetition-algorithm)  
15. We Made a New SRS System (WRS) for Games—Here's Why : r/Anki \- Reddit, accessed October 4, 2025, [https://www.reddit.com/r/Anki/comments/1iap6lm/we\_made\_a\_new\_srs\_system\_wrs\_for\_gamesheres\_why/](https://www.reddit.com/r/Anki/comments/1iap6lm/we_made_a_new_srs_system_wrs_for_gamesheres_why/)  
16. (PDF) Enhancing Learning and Engagement through Gamification of Student Response Systems \- ResearchGate, accessed October 4, 2025, [https://www.researchgate.net/publication/325077926\_Enhancing\_Learning\_and\_Engagement\_through\_Gamification\_of\_Student\_Response\_Systems](https://www.researchgate.net/publication/325077926_Enhancing_Learning_and_Engagement_through_Gamification_of_Student_Response_Systems)  
17. www.pominis.com, accessed October 4, 2025, [https://www.pominis.com/en/blog/branching-narratives-explained-designing-choices-that-matter\#:\~:text=Understanding%20Branching%20Narratives,multiple%20story%20paths%20and%20endings.](https://www.pominis.com/en/blog/branching-narratives-explained-designing-choices-that-matter#:~:text=Understanding%20Branching%20Narratives,multiple%20story%20paths%20and%20endings.)  
18. Structures of choice in narratives in Gamification and games \- æStranger \- aeStranger, accessed October 4, 2025, [https://aestranger.com/structures-choice-narratives-gamification-games/](https://aestranger.com/structures-choice-narratives-gamification-games/)  
19. From Linear Story Generation to Branching Story Graphs, accessed October 4, 2025, [https://faculty.cc.gatech.edu/\~riedl/pubs/riedl-aiide05.pdf](https://faculty.cc.gatech.edu/~riedl/pubs/riedl-aiide05.pdf)  
20. In a text based adventure game, how do I prevent long confusing conditional code?, accessed October 4, 2025, [https://stackoverflow.com/questions/39679878/in-a-text-based-adventure-game-how-do-i-prevent-long-confusing-conditional-code](https://stackoverflow.com/questions/39679878/in-a-text-based-adventure-game-how-do-i-prevent-long-confusing-conditional-code)  
21. What Video Games Have to Teach Us About Data Visualization | by Elijah Meeks \- Medium, accessed October 4, 2025, [https://medium.com/nightingale/what-video-games-have-to-teach-us-about-data-visualization-87c25ff7c62f](https://medium.com/nightingale/what-video-games-have-to-teach-us-about-data-visualization-87c25ff7c62f)  
22. Exploring New Trends in Puzzles and Games Education \- LearningMole, accessed October 4, 2025, [https://learningmole.com/new-trends-in-puzzles-and-games-education/](https://learningmole.com/new-trends-in-puzzles-and-games-education/)  
23. Analyzing the Influences of Puzzle Games on Learners' Learning, accessed October 4, 2025, [https://drpress.org/ojs/index.php/EHSS/article/download/12299/11978/12059](https://drpress.org/ojs/index.php/EHSS/article/download/12299/11978/12059)  
24. Overview of puzzle-based game design including virtual and physical objects, accessed October 4, 2025, [https://www.researchgate.net/figure/Overview-of-puzzle-based-game-design-including-virtual-and-physical-objects\_fig2\_265206701](https://www.researchgate.net/figure/Overview-of-puzzle-based-game-design-including-virtual-and-physical-objects_fig2_265206701)  
25. Consistent AI image generation style \- Figma Forum, accessed October 4, 2025, [https://forum.figma.com/ask-the-community-7/consistent-ai-image-generation-style-43431](https://forum.figma.com/ask-the-community-7/consistent-ai-image-generation-style-43431)  
26. How to write AI art prompts \[examples \+ templates\] \- Hootsuite Blog, accessed October 4, 2025, [https://blog.hootsuite.com/ai-art-prompts/](https://blog.hootsuite.com/ai-art-prompts/)  
27. Speech generation (text-to-speech) | Gemini API \- Google AI for Developers, accessed October 4, 2025, [https://ai.google.dev/gemini-api/docs/speech-generation](https://ai.google.dev/gemini-api/docs/speech-generation)  
28. Text to speech REST API \- Azure \- Microsoft Learn, accessed October 4, 2025, [https://learn.microsoft.com/en-us/azure/ai-services/speech-service/rest-text-to-speech](https://learn.microsoft.com/en-us/azure/ai-services/speech-service/rest-text-to-speech)  
29. Best Free Text-to-Speech AI APIs \- CAMB.AI, accessed October 4, 2025, [https://www.camb.ai/blog-post/best-free-text-to-speech-ai-apis](https://www.camb.ai/blog-post/best-free-text-to-speech-ai-apis)  
30. Home – Speechace | Pronunciation and fluency assessment via speech recognition, accessed October 4, 2025, [https://www.speechace.com/](https://www.speechace.com/)  
31. Phoneme-Level Visual Speech Recognition via Point-Visual Fusion and Language Model Reconstruction \- arXiv, accessed October 4, 2025, [https://arxiv.org/html/2507.18863v1](https://arxiv.org/html/2507.18863v1)  
32. karkirowle/relative\_phoneme\_analysis: Repository for phoneme analysis on word-level Kaldi/ESPNet ASR transcripts \- GitHub, accessed October 4, 2025, [https://github.com/karkirowle/relative\_phoneme\_analysis](https://github.com/karkirowle/relative_phoneme_analysis)  
33. Introduction to Automatic Speech Recognition (ASR) \- \- Maël Fabien, accessed October 4, 2025, [https://maelfabien.github.io/machinelearning/speech\_reco/](https://maelfabien.github.io/machinelearning/speech_reco/)  
34. Automatic Speech Recognition (ASR) Systems Applied to Pronunciation Assessment of L2 Spanish for Japanese Speakers \- MDPI, accessed October 4, 2025, [https://www.mdpi.com/2076-3417/11/15/6695](https://www.mdpi.com/2076-3417/11/15/6695)  
35. The Ultimate Guide to Organizing Your Next.js 15 Project Structure \- Wisp CMS, accessed October 4, 2025, [https://www.wisp.blog/blog/the-ultimate-guide-to-organizing-your-nextjs-15-project-structure](https://www.wisp.blog/blog/the-ultimate-guide-to-organizing-your-nextjs-15-project-structure)  
36. Best Practices for Organizing Your Next.js 15 2025 \- DEV Community, accessed October 4, 2025, [https://dev.to/bajrayejoon/best-practices-for-organizing-your-nextjs-15-2025-53ji](https://dev.to/bajrayejoon/best-practices-for-organizing-your-nextjs-15-2025-53ji)  
37. How to use the Next.js app directory: Complete guide and tutorial \- Contentful, accessed October 4, 2025, [https://www.contentful.com/blog/next-js-app-directory-guide-tutorial/](https://www.contentful.com/blog/next-js-app-directory-guide-tutorial/)  
38. Game UI design: the mechanics of fun experiences \- Justinmind, accessed October 4, 2025, [https://www.justinmind.com/ui-design/game](https://www.justinmind.com/ui-design/game)  
39. Game User Interface Design \- Meegle, accessed October 4, 2025, [https://www.meegle.com/en\_us/topics/gaming/game-user-interface-design](https://www.meegle.com/en_us/topics/gaming/game-user-interface-design)  
40. The 7 Best Software on Branching Story \- Said Hasyim, accessed October 4, 2025, [https://www.saidhasyim.com/best-product/software/branching-story/](https://www.saidhasyim.com/best-product/software/branching-story/)  
41. mdegans/weave: Branching story writing tool with generative AI \- GitHub, accessed October 4, 2025, [https://github.com/mdegans/weave](https://github.com/mdegans/weave)  
42. Next.js PWA offline capability with Service Worker, no extra package : r/Frontend \- Reddit, accessed October 4, 2025, [https://www.reddit.com/r/Frontend/comments/1mm6zgg/nextjs\_pwa\_offline\_capability\_with\_service\_worker/](https://www.reddit.com/r/Frontend/comments/1mm6zgg/nextjs_pwa_offline_capability_with_service_worker/)  
43. Offline First Support · vercel next.js · Discussion \#861 \- GitHub, accessed October 4, 2025, [https://github.com/vercel/next.js/discussions/861](https://github.com/vercel/next.js/discussions/861)  
44. Next.js PWA offline capability with Service Worker, no extra package \- A Drop In Calm, accessed October 4, 2025, [https://adropincalm.com/blog/nextjs-offline-service-worker/](https://adropincalm.com/blog/nextjs-offline-service-worker/)  
45. Building an Offline-First Next.js 15 App with App Router and Dynamic Routes \#82498, accessed October 4, 2025, [https://github.com/vercel/next.js/discussions/82498](https://github.com/vercel/next.js/discussions/82498)  
46. Offline data sync patterns \- ODC Documentation, accessed October 4, 2025, [https://success.outsystems.com/documentation/outsystems\_developer\_cloud/building\_apps/offline\_data\_synchronization\_in\_mobile\_apps/offline\_data\_sync\_patterns/](https://success.outsystems.com/documentation/outsystems_developer_cloud/building_apps/offline_data_synchronization_in_mobile_apps/offline_data_sync_patterns/)  
47. Offline Data Sync Patterns \- Overview (O11) \- OutSystems, accessed October 4, 2025, [https://www.outsystems.com/forge/component-overview/1638/offline-data-sync-patterns-o11](https://www.outsystems.com/forge/component-overview/1638/offline-data-sync-patterns-o11)  
48. Design Guidelines for Offline & Sync | Open Health Stack \- Google for Developers, accessed October 4, 2025, [https://developers.google.com/open-health-stack/design/offline-sync-guideline](https://developers.google.com/open-health-stack/design/offline-sync-guideline)  
49. Offline Data Synchronization in Mobile Apps \- Ideas2IT Technologies, accessed October 4, 2025, [https://www.ideas2it.com/blogs/offline-sync-native-apps](https://www.ideas2it.com/blogs/offline-sync-native-apps)