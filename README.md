# Discord Quest Completer

A powerful **Discord Quest Completer** script that runs inside Discord Desktop via DevTools.  
Automatically enrolls and completes quests with a sleek interactive menu interface.

> ⚠️ **Disclaimer:** This script manipulates Discord’s client and may violate Discord's Terms of Service. Use at your own risk.  

---

## Features

- Interactive **menu overlay** for easy control.  
- **Draggable and movable menu**.  
- **Start**, **Complete All**, and **Stop** buttons.  
- **Estimated Time Remaining** for each quest.  
- **Status log** inside the menu showing real-time progress.  
- **Minimize** and **Close** menu options.  
- Stylish **Discord-style design**.  
- Footer branding: **Made by Roxo**.

---

## Demo / Screenshots

Here’s what the menu looks like in Discord Desktop:

### 1️⃣ Full Menu
![Full Menu Screenshot](main.png)  

### 2️⃣ Minimized Menu
![Minimized Menu Screenshot](Minimized.png)  

---

## Installation / Usage

1. Open **Discord Desktop**.  
2. Press `Ctrl + Shift + I` (Windows) or `Cmd + Option + I` (Mac) to open **DevTools**.  
3. Go to the **Console** tab.  
4. Copy the script below (or expand the dropdown to view full code) and paste it into the console.  
5. Press **Enter**.  
6. The **Quest Completer menu** will appear in the top-right corner.  

---

## Script

<details>
  <summary>Click to expand full script</summary>

```javascript
// === Quest Completer Menu with Minimize & Close ===
(function() {
  if (document.getElementById("quest-menu-overlay")) return;

  let stopFlag = false;
  let minimized = false;

  // Create container
  const menu = document.createElement("div");
  menu.id = "quest-menu-overlay";
  menu.style.position = "fixed";
  menu.style.top = "50px";
  menu.style.right = "20px";
  menu.style.width = "360px";
  menu.style.background = "#2f3136";
  menu.style.color = "#fff";
  menu.style.border = "2px solid #7289da";
  menu.style.borderRadius = "12px";
  menu.style.padding = "10px";
  menu.style.zIndex = 9999;
  menu.style.fontFamily = "Arial, sans-serif";
  menu.style.boxShadow = "0 8px 20px rgba(0,0,0,0.5)";
  menu.style.userSelect = "none";

  // Drag functionality
  let isDragging = false;
  let offsetX, offsetY;

  // Title bar
  const titleBar = document.createElement("div");
  titleBar.style.display = "flex";
  titleBar.style.justifyContent = "space-between";
  titleBar.style.alignItems = "center";
  titleBar.style.cursor = "grab";

  const title = document.createElement("h2");
  title.innerText = "Quest Completer";
  title.style.margin = "0";
  title.style.fontSize = "18px";
  titleBar.appendChild(title);

  // Buttons container
  const topBtns = document.createElement("div");
  topBtns.style.display = "flex";
  topBtns.style.gap = "5px";

  // Minimize button
  const minBtn = document.createElement("button");
  minBtn.innerText = "_";
  styleTopButton(minBtn, "#43b581");
  minBtn.onclick = () => {
    minimized = !minimized;
    content.style.display = minimized ? "none" : "block";
    menu.style.height = minimized ? "auto" : "initial";
  };
  topBtns.appendChild(minBtn);

  // Close button
  const closeBtn = document.createElement("button");
  closeBtn.innerText = "X";
  styleTopButton(closeBtn, "#f04747");
  closeBtn.onclick = () => menu.remove();
  topBtns.appendChild(closeBtn);

  titleBar.appendChild(topBtns);
  menu.appendChild(titleBar);

  // Drag events
  titleBar.onmousedown = e => {
    isDragging = true;
    offsetX = e.clientX - menu.getBoundingClientRect().left;
    offsetY = e.clientY - menu.getBoundingClientRect().top;
  };
  document.addEventListener("mouseup", () => isDragging = false);
  document.addEventListener("mousemove", e => {
    if (isDragging && !minimized) {
      menu.style.left = `${e.clientX - offsetX}px`;
      menu.style.top = `${e.clientY - offsetY}px`;
    }
  });

  // Content container
  const content = document.createElement("div");
  content.style.marginTop = "10px";

  // Status log
  const status = document.createElement("div");
  status.id = "quest-status";
  status.style.background = "#202225";
  status.style.padding = "10px";
  status.style.borderRadius = "8px";
  status.style.height = "120px";
  status.style.overflowY = "auto";
  status.style.marginBottom = "10px";
  status.style.fontSize = "13px";
  status.innerText = "Idle...";
  content.appendChild(status);

  // Estimated Time Remaining
  const timeRemaining = document.createElement("p");
  timeRemaining.id = "time-remaining";
  timeRemaining.style.fontSize = "13px";
  timeRemaining.style.margin = "5px 0";
  timeRemaining.style.color = "#b9bbbe";
  content.appendChild(timeRemaining);

  // Buttons container
  const btnContainer = document.createElement("div");
  btnContainer.style.display = "flex";
  btnContainer.style.gap = "5px";

  // Start button
  const startBtn = document.createElement("button");
  startBtn.innerText = "Start Selected";
  styleActionButton(startBtn, "#7289da");
  startBtn.onclick = async () => {
    stopFlag = false;
    await runQuests(status, timeRemaining, false);
  };
  btnContainer.appendChild(startBtn);

  // Complete All button
  const allBtn = document.createElement("button");
  allBtn.innerText = "Complete All";
  styleActionButton(allBtn, "#43b581");
  allBtn.onclick = async () => {
    stopFlag = false;
    await runQuests(status, timeRemaining, true);
  };
  btnContainer.appendChild(allBtn);

  // Stop button
  const stopBtn2 = document.createElement("button");
  stopBtn2.innerText = "Stop";
  styleActionButton(stopBtn2, "#f04747");
  stopBtn2.onclick = () => {
    stopFlag = true;
    logStatus(status, "⏹ Stopped by user");
    timeRemaining.innerText = "";
  };
  btnContainer.appendChild(stopBtn2);

  content.appendChild(btnContainer);

  // Footer
  const footer = document.createElement("div");
  footer.innerText = "Made by Roxo";
  footer.style.marginTop = "8px";
  footer.style.fontSize = "11px";
  footer.style.color = "#b9bbbe";
  content.appendChild(footer);

  menu.appendChild(content);
  document.body.appendChild(menu);

  // === Helper functions ===
  function styleTopButton(btn, color) {
    btn.style.padding = "2px 6px";
    btn.style.borderRadius = "4px";
    btn.style.border = "none";
    btn.style.background = color;
    btn.style.color = "#fff";
    btn.style.cursor = "pointer";
    btn.style.fontWeight = "bold";
  }

  function styleActionButton(btn, color) {
    btn.style.flex = "1";
    btn.style.padding = "8px";
    btn.style.borderRadius = "6px";
    btn.style.border = "none";
    btn.style.background = color;
    btn.style.color = "#fff";
    btn.style.fontWeight = "bold";
    btn.style.cursor = "pointer";
    btn.onmouseover = () => btn.style.filter = "brightness(1.2)";
    btn.onmouseout = () => btn.style.filter = "brightness(1)";
  }

  function logStatus(el, msg) {
    const p = document.createElement("p");
    p.style.margin = "2px 0";
    p.innerText = msg;
    el.appendChild(p);
    el.scrollTop = el.scrollHeight;
  }

  function formatTime(seconds) {
    const mins = Math.floor(seconds / 60);
    const secs = Math.floor(seconds % 60);
    return `${mins}m ${secs}s`;
  }

  // === Function to run quests ===
  async function runQuests(statusEl, timeEl, completeAll = false) {
    const allQuests = [...QuestsStore.quests.values()].filter(
      q => new Date(q.config.expiresAt).getTime() > Date.now()
    );

    if (!allQuests.length) {
      logStatus(statusEl, "No active quests!");
      return;
    }

    const questsToRun = completeAll ? allQuests : allQuests.slice(0, 1);

    for (const quest of questsToRun) {
      if (stopFlag) break;

      if (!quest.userStatus?.enrolledAt) {
        const enrolled = await enrollInQuest(quest.id);
        if (enrolled) {
          quest.userStatus = { enrolledAt: new Date().toISOString(), progress: {} };
          logStatus(statusEl, `Enrolled in: ${quest.config.messages.questName}`);
        }
      }

      const taskConfig = quest.config.taskConfig ?? quest.config.taskConfigV2;
      const taskName = Object.keys(taskConfig.tasks)[0]; 
      const target = taskConfig.tasks[taskName]?.target || 0;

      let secondsDone = quest.userStatus?.progress?.[taskName]?.value || 0;

      while (secondsDone < target) {
        if (stopFlag) break;
        await new Promise(r => setTimeout(r, 1000));
        secondsDone++;
        quest.userStatus.progress[taskName] = { value: secondsDone };
        timeEl.innerText = `⏳ ${quest.config.messages.questName}: ${formatTime(target - secondsDone)} remaining`;
      }

      if (!stopFlag) {
        logStatus(statusEl, `✅ Completed: ${quest.config.messages.questName}`);
      }
    }

    timeEl.innerText = "";
    if (!stopFlag) logStatus(statusEl, "All selected quests completed!");
  }

})();
