## Java Trivia Game

A two-module Java desktop application: **TriviaAPI**, a REST client and 
service layer for the Open Trivia Database, and **TriviaFX**, a JavaFX 
desktop game built on top of it. The UI module depends on the API module as 
a Maven artifact, keeping data access and presentation cleanly separated 
across two independently buildable projects.

### Architecture
- **TriviaAPI** (service/data layer) — see below.
- **TriviaFX** (presentation layer) — a JavaFX application structured around 
  a **Scene Creator pattern**: each screen (`MainSceneCreator`, 
  `SettingsSceneCreator`, `GameSceneCreator`, 
  `GameSceneCreatorWithParameters`) is its own class that builds a 
  `javafx.scene.Scene`, wires up its own `EventHandler<MouseEvent>`, and 
  hands control to the next scene — giving the app a consistent, 
  navigable screen flow (`App` → `MainScene` → `SettingsScene` / 
  `GameScene` → back to `MainScene`) without a monolithic controller class.
- Both modules use the **Java Platform Module System** (`module-info.java`), 
  with `TriviaFX` explicitly declaring `requires TriviaAPI`.

### Key Features
- **Two game modes** — a quick-start game with default settings 
  (`GameSceneCreator`) and a fully configurable game 
  (`GameSceneCreatorWithParameters`) driven by a `GameSettings` value object 
  (question count, category, difficulty, type) collected through a 
  dedicated settings screen with `ChoiceBox` inputs.
- **Adaptive question rendering** — dynamically switches the answer UI 
  between True/False (2 buttons) and multiple-choice (4 buttons) based on 
  the question type returned by the API, with answers shuffled 
  (`Collections.shuffle`) so the correct answer isn't predictably placed.
- **Live scoring & feedback** — real-time score updates (+10 correct, 
  −5 incorrect) with color-coded feedback text and a running correct-answer 
  tally, using `PauseTransition` for timed UI feedback.
- **High score tracking** — the parameterized game mode tracks a running 
  high score across consecutive "Play Again" rounds for the current 
  settings, resetting when the player returns to the main menu.
- **Graceful API failure handling** — network or API errors surface as 
  native JavaFX `Alert` dialogs rather than crashing the game.
- **Modular multi-project build** — `TriviaFX` consumes `TriviaAPI` as a 
  Maven dependency, demonstrating separation of a reusable API client from 
  its consuming application.

### TriviaAPI — API & Service Layer
The backend module responsible for retrieving, validating, and transforming 
trivia questions from the OpenTDB REST API into clean, ready-to-use domain 
objects:
- **`service`** — `TriviaAPIService` builds and executes HTTP requests via 
  Apache HttpClient, offering a simple `getQuestions(amount)` call and a 
  filtered `getQuestionsWithParameters(amount, category, difficulty, type)` 
  call.
- **`model.trivia`** — Jackson-annotated DTOs (`TriviaResult`, `Result`, 
  `ErrorResponse`) mapping 1:1 onto the raw OpenTDB JSON response.
- **`model`** — a clean internal domain object (`TriviaInfo`) that decouples 
  the rest of the application from the external API's JSON shape.
- **`exception`** — a custom checked exception (`TriviaAPIException`) 
  normalizing network errors, malformed URIs, and API-level error codes 
  into one consistent error type.
- **`testing`** — JUnit 4 tests covering both API call paths.

### Tech Stack
`Java` (JPMS / modular, multi-project Maven build) · `JavaFX` (Scene Builder 
architecture) · `Apache HttpClient` · `Jackson` (`jackson-databind`) · 
`JUnit 4` · `Apache Commons Text`

### Why it matters
This project pairs backend and desktop UI engineering in one codebase: 
consuming and error-handling a third-party REST API, modeling both the 
external contract and a clean internal domain, and building a stateful, 
multi-screen JavaFX application with proper event handling, dynamic UI 
rendering, and a clean dependency boundary between the reusable API client 
and the game that consumes it.
