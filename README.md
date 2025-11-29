# ⚡️ Spark_Labs: Sentinel
**Turn your Shelly BLU Button into a Secure Keypad.**
*Current Release: v1.0 (BETA) | Target: Shelly Gen3 & Gen4*

---

### 🛡 Project Overview
**Sentinel** is a script-based security framework that transforms a standard **Shelly BLU Button** into a secure, multi-factor authentication keypad.

Unlike standard smart button automations—where a single click grants immediate access—Sentinel requires **Pattern Recognition**. Users must input a specific sequence of clicks (e.g., *Double, Single, Long*) to trigger the relay. This ensures that **possessing the button is not enough to unlock the door; you must also know the code.**

### 🧠 System Architecture
The system operates on a **Distributed Logic** model using distinct scripts running on a Shelly Gen3/Gen4 device:

| Component | Script Name | Function |
| :--- | :--- | :--- |
| **The Ear** | `Spark_Bridge` | **High-Frequency BLE Scanner.** Listens for raw BLE packets, filters for specific MAC addresses, ignores duplicates, and forwards clean events to the event bus. |
| **The Brain** | `Spark_Brain` | **State Machine & Logic Engine.** Buffers inputs into a sequence array, compares them against a user database, handles timeouts, enforces security lockouts, and triggers the physical relay. |
| **The Hand** | `Spark_Installer` | **UI Generator.** A one-time utility that programmatically builds the Virtual Components (UI) in the Shelly App, ensuring a consistent interface for status, history, and feedback. |
| **The Ghost** | `Spark_Simulator` | **QA Tool.** A simulation script that injects synthetic events (like button presses and intruders) to test the system logic without physically standing at the door. |

### 🛡 Key Capabilities
* **MFA (Possession + Knowledge):** Converts a simple Bluetooth button into a "Something you have" + "Something you know" security device.
* **User-Specific Access:** Supports multiple user profiles (e.g., Dad, Mum, Kids), each with unique click-sequences and logging.
* **Brute Force Protection:** Temporarily locks the system after 3 failed attempts to prevent random clicking.
* **Smart Logging:** Tracks who opened the door and when, distinguishing between specific users and manual key usage (detected via door sensor logic).
* **Visual Feedback:** Uses Shelly Virtual Components to provide real-time status updates (🔒 Locked, ⏰ Input Active, ✅ Success) directly in the Shelly Smart Control App.

### 🔌 Hardware Requirements
* **Host:** Shelly Gen3 or Gen4 Device (Running the scripting engine).
* **Input:** Shelly BLU Button Tough (recommended for durability/buzzer).
* **Sensor:** Shelly BLU Door Sensor (Optional, for "Manual Entry" detection).

---

### 📖 The Story: Why I Built This
I didn't build this for **"random intruders."** I built this because I have three 11-year-old daughters. They grew up living in a Shelly-powered smart home and expect convenience 🤣—but they also like to destroy and lose stuff, and push my automations to the limit.

For years, we had a simple Shelly Gen 1 v3, but we found out early on that it wasn't secure enough. The usage had to be restricted because **the kids figured out they could unlock the front door by shouting at Google from the mailbox outside!**

On top of that, my wife was constantly complaining that we had to chase them to know if they arrived home safely—and one of my daughters in particular would lose her head if it wasn't attached.

I needed a solution that was:
1.  **Durable:** The **Shelly BLU Button Tough** (it has a buzzer to find lost keys and stands up to abuse).
2.  **Secure:** Solving the **"Possession = Access"** problem. Giving them a standard button was a recipe for disaster (accidental presses and doors left open).
3.  **Smart:** Converting the button into a **secure keypad** using the Gen3 scripting engine.

---

### 🛠 Credits & References
This project is **built on the shoulders of giants**. A massive thank you to the guys at **Shelly** for their excellent device software API and script libraries.

**Special thanks to the Shelly Academy instructors for the excellent courses, content, and advanced knowledge that made solutions like this possible.**

* **Script A (The Bridge):** Adapted from the official [Shelly Script Examples](https://github.com/ALLTERCO/shelly-script-examples) (BLE Scanner). It handles **high-speed scanning** and filters raw packets.
* **Script C (The Installer):** Logic adapted from the `create-demo-virtual-components.js` found in the Shelly Academy/Script Examples.

---

### ⚠️ WARNINGS & DISCLAIMER
> **READ CAREFULLY:** I am not a professional developer, and running this script entrusts the security of your personal property at your own risk. We use this on an electronic lock when all are away, and the door is still locked with a key.

* ⚡️ **DANGER:** Electricity is deadly. If you do not have the knowledge or experience to install Shelly devices, **seek professional advice** ⚡️
* **DANGER:** The installation process will **trigger the output of the relay**. It is **strongly advised to disconnect the O terminal** before starting configuration.
* **DANGER:** When controlling motors remotely (e.g., garage doors), it is paramount that **obstacle avoidance sensors override the Shelly** to prevent injury or damage to personal property.
* If running on an existing device, ensure **all virtual components or groups are removed** before starting.

---

### 📦 Installation Guide
*Follow this exact order to ensure a clean installation.*

#### Phase 1: Zero State (Hardware Prep)
* **Goal:** Get your devices onto the network.
* **Hardware:** 1x **Shelly Gen3/Gen4 Device**, 1x **Shelly BLU Button Tough** (or more), 1x **Shelly BLU Door Sensor**.
* **Step 1:** Add your Shelly Gen3/4 relay and **update firmware** immediately.
* **Step 2: Safety Configs:**
    * Set **"Auto Off"** on the Relay Timer to **1 second**. This ensures the door strike doesn't stay open.
    * Go to Input/Output settings on the Relay and set **Power On Default** to **"Off"**. This prevents the lock from opening after a power outage.
* **Step 3:** Pair your BLU Buttons and Door Sensor. **Update their Firmware** as well.
* **Step 4:** **Crucial:** Record the **MAC addresses** of your BLU Buttons and door sensor (e.g., `aa:bb:cc...`).

#### Phase 2: The Bridge (Script A)
* **Goal:** Establish the hardware link.
* **Step 1:** Create a new script named `Spark_Bridge`.
* **Step 2:** Paste the code from `sentinel_script_a_bridge.js`.
* **Step 3:** Update **`CONFIG.MACS`** with the addresses you saved in Phase 1.
* **Step 4:** **RUN** the script. Open the Console. Press your buttons. You should see logs like `>>> BRIDGE: single_push from aa:bb:cc...`.
* **Step 5:** **STOP** the script.

#### Phase 3: The Brain (Script B)
* **Goal:** Configure the logic and users.
* **Step 1:** Create a new script named `Spark_Brain`.
* **Step 2:** Paste the code from `sentinel_script_b_brain.js`.
* **Step 3: Configuration:** Update **`CONFIG.USERS`**.
    * Give your family names (e.g., **"DAD"**). **Note: Do not assign the emoji now**.
    * Assign their specific button sequences (e.g., `["double_push", "single_push", "long_push"]`).
    * Assign MAC addresses for each user button.
* **Step 4:** **SAVE** only. **Do not run this yet.**

#### Phase 4: The Installer (Script C)
* **Goal:** Auto-generate the Interface.
* **Step 1:** Create a new script named `Spark_Installer`.
* **Step 2:** Paste the code from `sentinel_script_c_installer.js`.
* **Step 3:** **User Display Configuration:** Inside Script C, replace generic usernames with your desired **display text** (e.g., `"User 1": "🧔🏻 DAD"`). The user text in Script B and C should be **consistent and case-sensitive**.
* **Step 4:** **RUN** the script. You will see the components being generated live.
* **Step 5:** **DELETE** Script C. It is no longer needed.

#### Phase 5: UI Refinement (The "Pro" Look)
* **Goal:** Make it look like a real App.
* **Step 1:** In the Shelly App, go to **Components** and find the new Group called **"Door Control"**.
* **Step 2:** Click the **Settings** (cog) icon.
* **Step 3:** Select **"Extract virtual group as device"**.
    * *Note: Free accounts are limited to **1 Virtual Device**. Premium subscriptions allow more*.
* **Step 4:** Open the new Device Card. Go to **App Settings (2 cogs) -> Customize device card**.
* **Step 5:** **Manual Customization:** Select the **Small Parameter** slot and reorder the components for optimal display:
    * **01 Status:** 🔒
    * **02 Door state:**
    * **03 Progress (Visuals):**
    * **04 User (User History):** ✋️
    * **05 Last opened (Timestamp):** 18:25 | T

#### Phase 6: The Simulator (Script D)
*This step can be skipped for manual testing but is a great way to test your UI and user names fit.*
* **Goal:** Verify everything works.
* **Step 1:** **Start Script B (Brain)** before starting the simulator.
* **Step 2:** Create a script named `Spark_Simulator`.
* **Step 3:** Paste the code from `sentinel_script_d_simulator.js`.
* **Step 4:** Paste the MAC addresses from Phase 1 to ensure the simulator targets the correct virtual users.
* **Step 5:** **RUN** Script D.
* **Step 6:** Watch the **"Ghost Demo."** The simulator will auto-type codes, trigger lockouts, and log entries.
* **Warning:** **STOP** the script immediately after testing. **Do not** set it for auto-run.

#### Phase 7: Go Live
* **Step 1:** Start **Script A (Bridge)**.
* **Step 2:** Start **Script B (Brain)**.
* **Step 3:** Enable **"Run on Startup"** for both.
* **Step 4:** Test it manually with your physical button! Remember, you must be within range of the device.

#### Phase 8: Automation & Scenes (Optional)
*Requires Shelly Premium Subscription*
You can use the **"User History"** virtual component to trigger Shelly Scenes for advanced notifications.
1.  Create a Scene in the Shelly App.
2.  **When:** Select "Component Property Changed" -> **User History**.
3.  **Condition:** Set the condition to match specific status updates.
    * **"Honey I'm Home!"** -> Trigger when status is **"🧔🏻 DAD"**.
    * **Child Safety:** Trigger notification when status is **"👤 Kid1"**.
    * **Security Alert:** Trigger an alarm if status becomes **"⛔ Lockout"**.
    * **Manual Entry:** Log "Somebody opened the door" if status is **"🖐️ Manual"**.

---

### 📝 Release Notes: v1.0 (BETA)
**Release Date:** November 2025
**Status:** Beta Ready

This is the definitive **"Static" version**, designed for maximum security and stability without requiring external databases or complex configuration.

**Latest Features:**
* **Event Storm Protection:** Implemented a **FIFO Buffer** (Reverse Queue) to handle rapid typing without crashing the event loop.
* **One-Click Installer:** Added `Spark_Installer` (Script C) to auto-generate Virtual Components.
* **Smart History:** Logs intelligent timestamps (`Today | 14:00` vs `25/11 | 14:00`).
* **Manual Detection:** If the door opens without a code (e.g., using a physical key), it logs as **"🖐️ Manual"**.
* **Multi-Factor Security:** Includes **Possession, Knowledge, and Location (RSSI Floor)** checks.

**Roadmap (Coming Soon):**
* **v2 (The Database Update):** Migrating users from script code to Shelly KVS (`access_db`).
* **v3 (Duress Protocol):** A "Panic Code" that opens the door while silently triggering a UDP Network Alarm.
* **Button Feedback:** Triggering the buzzer on the buttons to provide feedback on failed attempts.
* **Advanced Context:** Key hook tracking using button RSSI and directional awareness using the BLU Motion sensor.
* **Wired Options:** Wired door sensor version or configuration option and input switch/sensor configuration.
