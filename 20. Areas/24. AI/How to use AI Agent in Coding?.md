---
tags:
  - ai
  - ai/agent
---
#### Concept: Building Software with an AI Agent

Think of me as your AI pair programmer. You are the project manager and visionary; I am the hands-on developer who writes and executes the code based on your direction. The process is a conversation where you guide the development iteratively.

---

#### Step 1: Define Your Application (The "What")

This is the most important step. You need to tell me what you want to build. A good starting prompt includes:

*   **Core Purpose:** What is the main goal of your software? (e.g., "a to-do list app," "a personal blog," "a weather dashboard").
*   **Key Features:** List the essential features. (e.g., "Users should be able to add tasks, mark them as complete, and delete them").
*   **Technology Preference (Optional):** If you have a preference, mention it. (e.g., "Let's build this with React and Node.js"). If you don't, I will propose a modern, standard stack.

**Your Action:** Write a clear, descriptive prompt.
**Example:** *"I want to create a simple web-based calculator. It should have buttons for numbers 0-9, basic arithmetic operations (+, -, *, /), a clear button, and an equals button. Let's use HTML, CSS, and JavaScript."*

#### Step 2: Review and Approve the Plan (The "How")

Based on your request, I will propose a high-level plan. This will include:

*   The technology stack I'll use.
*   The file structure I'll create.
*   A summary of the core features and my approach to building them.

**Your Action:** Review the plan. If it looks good, approve it. If you want changes, let me know.
**Example AI Response:** *"Okay, I will build a calculator using HTML for the structure, CSS for styling, and JavaScript for the logic. I'll start by creating three files: `index.html`, `style.css`, and `script.js`. First, I'll build the UI, then I'll implement the calculation logic. Does that sound good?"*

#### Step 3: Project Scaffolding (The Foundation)

Once you approve, I will begin setting up the project. This involves:

*   Creating the main project directory.
*   Using shell commands to initialize the project (e.g., `npm init`).
*   Creating the initial folder structure and empty files (e.g., `index.html`, `style.css`, `script.js`).

**Your Action:** Simply watch and confirm. I will explain the commands I'm running before I execute them.

#### Step 4: Iterative Feature Implementation (The Building Blocks)

We will now build the application piece by piece. You guide the process by asking for one feature at a time.

**Your Action:** Request a specific, small feature.
**Example:** *"Okay, let's start. Create the HTML structure for the calculator in `index.html`."*

**My Role:** I will use my tools (`write_file`, `replace`) to write the necessary code for that feature. I will show you the code I'm adding. We'll continue this for all features, from the UI to the backend logic.

#### Step 5: Testing and Verification (Quality Assurance)

After implementing a feature, it's crucial to test it.

**Your Action:** Ask me to run the application or write a test.
**Example:** *"The calculator UI is done. Now, let's add the JavaScript logic to make the number buttons work."* or later, *"Run the application so I can see how it looks."*

**My Role:** I will write the code, start the development server using `run_shell_command` (e.g., `npm start`), and give you the URL to view the app. If something is wrong, you tell me, and I'll fix it.

#### Step 6: Review and Refine (The Feedback Loop)

This is a continuous cycle.

1.  **You:** Request a feature.
2.  **I:** Implement it.
3.  **You:** Review it and provide feedback (e.g., "The display is too small," or "The addition logic is wrong").
4.  **I:** Refine the implementation based on your feedback.

We repeat this loop until the software meets your requirements. By breaking the large goal of "creating software" into these small, manageable steps, you can direct the entire development process effectively, even as a beginner.

---

