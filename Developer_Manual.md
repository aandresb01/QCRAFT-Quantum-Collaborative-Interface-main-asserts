# QCraft Developer Manual

This document details the necessary steps to deploy and run the QCraft development environment.

## 1. Clone the Repository

```bash
git clone https://github.com/aandresb01/QCRAFT-Quantum-Collaborative-Interface-main.git
```

## 2. Project Deployment

The system consists of four main components that must run simultaneously.

### Django Backend (Authentication & User Management)

1. Navigate to the Django project folder.
2. Ensure **Python 3.10.11+** is installed.
3. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```
4. Start the server:

   ```bash
   python manage.py runserver
   ```
5. The backend will be available at `http://localhost:8000`.

---

### Spring Boot Backend (WebSocket & Collaborative Editing)

1. Navigate to the Spring Boot project folder.
2. Ensure **Java (JDK 17+)** and **Maven** are installed.
3. Compile and run the server:

   ```bash
   cd quirkShareSpring/
   mvn spring-boot:run
   ```
4. This service will handle WebSockets for collaborative circuit editing.

---

### React Frontend - QCraft (Main Interface with Auth0)

1. Navigate to the project folder:

   ```bash
   cd UpdateQuirk/Auth0/
   ```

2. Ensure **Node.js** is installed.

3. Install dependencies:

   ```bash
   npm install
   ```

4. **Auth0 Configuration**:

   * Create a new *Single Page Application* in Auth0.
   * > ⚠️ Add `http://localhost:3000` to **Allowed Callback URLs**.
   * Create the configuration file at `UpdateQuirk/Auth0/src/auth_config.json` with the following content:

   ```json
   {
     "domain": "example.us.auth0.com",
     "clientId": "example",
     "audience": "https://example.us.auth0.com/api/v2/"
   }
   ```

   *(Domain and ClientID are in **Settings**; Audience is in **APIs**)*

5. Start the server:

   ```bash
   npm start
   ```

6. The main interface will be available at `http://localhost:3000`.

---

### React Frontend - Quirk (Circuit Visualization)

1. Navigate to the project folder:

   ```bash
   cd UpdateQuirk/Quirk/out
   ```
2. Install dependencies:

   ```bash
   npm install
   ```
3. Start the development server:

   ```bash
   npm run dev
   ```
4. **Important**:

   * Delete the `quirk.html` file.
   * Rename `quirk_util.html` -> `quirk.html`.
   * Refresh the page.
5. This environment allows real-time visualization of circuits.

---

## Final Note

> 📝 Once all four servers (Django, Spring, QCraft, Quirk) are running, the system is ready for collaborative real-time use.

---

## 3. Usage: Debug Mode

In the **My Circuits** section, an additional button appears next to the play button, represented by a bug icon 🐞. Clicking it opens the circuit in **DEBUG Mode**.

This mode enables additional tools for step-by-step circuit analysis:

* **Step Navigation**: Buttons **Previous Step** and **Next Step** are added at the top, allowing navigation through execution states.
* **Visual Tracking**: Step tracking is highlighted in gray, showing:

  * The results of each circuit column, in addition to the final outcome.
* **Dynamic Update**: The tracking state updates dynamically as you move forward or backward using the buttons.

### Usage Examples

* **CASE 1**: Results are already saved in the database; no need to run the circuit to view outcomes.

![VIDEO SAMPLE 1](./gifs/IDyaGuardado.gif)

* **CASE 2**: The circuit has been modified, so it must be executed to see the new results.

![VIDEO SAMPLE 2](./gifs/NuevoCircuito.gif)

### Using Breakpoints

Debug mode includes a **Breakpoints** functionality that allows stopping circuit execution at a specific column.

*   **Visual Indicator**: In Debug mode, you will see gray circles at the bottom of each circuit column.
*   **Activation**: Clicking on one of these circles will turn it **red**, indicating that a breakpoint has been set at that column.
*   **Behavior**:
    *   When the circuit is executed with an active breakpoint, execution will stop at the selected column.
    *   Results sent to the backend and displayed in the interface will be truncated up to the breakpoint point (inclusive).
    *   This is useful for isolating and analyzing specific parts of a complex circuit without running it entirely.

[Watch Video: Debug Mode Breakpoints](./gifs/ModoDebug_BreakPoints.mp4)

### Using Assertions

Debug mode includes an **Assertions** functionality that allows verifying if the obtained results match the developer's expectations.

*   **Input Field**: At the bottom of the Debug screen, a text field appears with the message *"Write the expected results (Format: 000, 100, 1001 ...)"*.
*   **Flexible Format**: You can enter the expected results as a list of bitstrings separated by commas (e.g., `000, 100`). The system will automatically pad with leading zeros if necessary to match the circuit's register length.
*   **Visual Feedback**:
    *   🟢 **Green**: The user input exactly matches the most probable results in the current graph.
    *   🔴 **Red**: There is a discrepancy between the input and the obtained results.
*   **Dynamic Verification**: Validation is performed automatically every time you change steps (columns) in the circuit, allowing for guided debugging based on expectations.

[Watch Video: Debug Mode Assertions](./gifs/ModoDebug_Assertions.mp4)

