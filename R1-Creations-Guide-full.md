# Rabbit R1 Creations — Complete Guide
> Everything you need to build, host, and install R1 Creations for free

---

## What Are R1 Creations?

R1 Creations are custom AI-generated mini-applications built specifically for the Rabbit R1 device. They function as specialized tools, games, and utilities that tap into the R1's unique hardware — including the push-to-talk (PTT) button, scroll wheel, microphone, rotating camera, speaker, and accelerometer. Creations were introduced as part of rabbitOS 2 in September 2025.

---

## The 3 Methods At a Glance

| Goal | Method | Cost |
|------|--------|------|
| Try someone else's creation | Gallery / Public Tab on R1 | Free |
| Build something simple, no coding | Speak to R1 (Intern) | 3 free tasks, then paid |
| Build a custom app, full control | SDK + GitHub Pages + QR code | Free (coding required) |
| Use AI to write the code | SDK + Claude/ChatGPT + host + QR | Free |

---

## Method 1 — Speak to Your R1 (Uses Intern Tasks)

This is the easiest way — you literally just describe what you want to build and the R1 builds it. You get **3 free Intern Tasks** when you create a new account.

### Intern Task Pricing

| Plan | Price | Tasks Included |
|------|-------|----------------|
| Free (new accounts) | $0 | 3 tasks |
| Pay-as-you-go | ~$10/task | 3 for ~$30 |
| Subscription | ~$2.33/task | 30 for ~$70 |

Pricing includes hosting — your creation is deployed and live immediately.

### Steps

1. Navigate to the **Creations card** on your R1 (scroll through your card stack)
2. Tap **"Create with Intern"**
3. **Talk to the agent** — describe exactly what you want in as much detail as possible (you only get one task credit per prompt, so be thorough). Example: *"Build me a fishing tide and weather tracker for the California Delta."*
4. The Intern agent builds it — Rabbit Intern acts like a project manager with specialized sub-agents
5. You receive:
   - A **QR code** to install the creation on your R1
   - A **shareable link** to the hosted app
   - A **source code download** of everything it built

> **Tip:** Write out your full feature list before speaking. Pack all requirements into one detailed prompt.

---

## Method 2 — Build for FREE Using the SDK

This is the developer path. You code the creation yourself (or use an AI like Claude/ChatGPT to write it), host it for free, and install via QR code — **zero cost**.

### What You Need

- A code editor (VS Code, or any text editor)
- A free GitHub account → https://github.com
- The official Rabbit R1 Creations SDK → https://github.com/rabbit-hmi-oss/creations-sdk

---

### PART 1 — Get the SDK

1. Go to **https://github.com/rabbit-hmi-oss/creations-sdk**
2. Click the green **"Code"** button → **"Download ZIP"**
   - Or clone it: `git clone https://github.com/rabbit-hmi-oss/creations-sdk`
3. Inside the repo you'll find two important folders:
   - **`plugin-demo`** — template showing all R1 hardware features (PTT, scroll wheel, camera, mic, etc.)
   - **`qr`** — a browser-based tool to generate install QR codes for your R1
4. Read through the docs to understand what APIs and hardware hooks are available

---

### PART 2 — Build Your Creation

Your creation is a **hosted JavaScript web app** (HTML/CSS/JS) optimized for the R1's small screen.

**Technical constraints:**
- Screen size: **240×282 pixels** (portrait)
- Avoid heavy graphics or complex animations
- No large file storage
- Only one creation can run at a time

**Build options:**
- **Code it yourself** — use the `plugin-demo` folder as your starting template
- **Use an AI tool** — paste the SDK docs into Claude Code, Cursor, or ChatGPT and describe your app. The community recommends this approach to avoid spending Intern tasks.

---

### PART 3 — Set Up GitHub (Free Hosting)

#### Step 1 — Create a Free GitHub Account

1. Go to **https://github.com** and click **Sign up**
2. Enter your email, create a password, and pick a username
3. Solve the bot verification puzzle, then **verify your email** with the code GitHub sends you
4. Select the **Free plan** — everything you need is on the free tier

#### Step 2 — Create a New Repository

1. Click the **"+"** button (top-right) → **"New repository"**
2. Name it something like `my-r1-creation` (lowercase, no spaces)
3. Set visibility to **Public** (required for free GitHub Pages hosting)
4. Check **"Add a README file"** so the repo initializes
5. Click **"Create repository"**

#### Step 3 — Upload Your Creation Files

**Browser Upload (Easiest):**
1. Inside your repo, click **"Add file"** → **"Upload files"**
2. Drag and drop your HTML/JS/CSS files from the SDK `plugin-demo` folder
3. **Make sure your main file is named `index.html`** — GitHub Pages requires this
4. Click **"Commit changes"**

**Git CLI (Faster for future updates):**
```bash
git init
git add .
git commit -m "Initial commit"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/my-r1-creation.git
git push -u origin main
```
Replace `YOUR-USERNAME` and `my-r1-creation` with your actual username and repo name.

#### Step 4 — Enable GitHub Pages (Free Hosting)

1. Go to your repository → click the **"Settings"** tab
2. Scroll down the left sidebar → click **"Pages"**
3. Under **Source**, select **"Deploy from a branch"**
4. Set branch to **`main`** and folder to **`/ (root)`**
5. Click **Save**
6. Wait **1–2 minutes** — GitHub gives you a live URL like:
   `https://YOUR-USERNAME.github.io/my-r1-creation/`

> **Important:** Add a blank file called `.nojekyll` to your repo root so GitHub doesn't process your JS files through its Jekyll builder. Create a new file, name it `.nojekyll`, leave it empty, and commit it.

#### Step 5 — Test Your Live URL

- Open the GitHub Pages URL in a browser
- Resize the browser window to roughly **240×282 pixels** to simulate the R1 screen
- Confirm everything loads correctly before moving on

---

### PART 4 — Generate the QR Code

The SDK's **`qr`** folder is a self-contained browser tool that creates R1-compatible install QR codes. No server needed — it runs entirely offline in your browser.

#### How to Run It

1. Open the **`qr`** subfolder inside the SDK you downloaded
2. Find `index.html` inside it — **double-click it** to open it in your browser (Chrome, Firefox, Edge — any will work)
3. Fill in the form fields:
   - **Name** — what your creation will be called on the R1 (e.g., `Delta Weather Tool`)
   - **Description** — a short sentence about what it does
   - **URL** — paste in your **GitHub Pages live URL** (e.g., `https://yourusername.github.io/my-r1-creation/`)
4. Click **Generate** — a scannable QR code appears immediately on the page

> The QR code simply encodes your hosted URL + name + description. Once scanned, the R1 loads your creation directly from GitHub Pages every time you open it.

#### Alternative: Submit to the Public Gallery

If you want your creation publicly shareable, submit it to **https://rabbit.tech/creations**. Each listed creation gets its own QR code that anyone with an R1 can scan and install for free.

---

### PART 5 — Install on Your R1

1. **Pick up your R1** and scroll through your card stack to the **Creations card**
2. Tap to open it — you'll see **Create** and **Public** tabs
3. Tap the **Create tab**
4. Scroll down and tap **"Add via QR code"**
5. Your R1's rotating camera (the "rabbit eye") activates in scan mode
6. **Hold your R1 up toward your computer screen** with the camera facing the QR code — about 6–12 inches away, keep it steady
7. The R1 automatically detects and reads the QR code — no button press needed
8. A confirmation message appears and your creation installs instantly
9. **Scroll to the bottom of your card stack** — your new creation is there as its own card

---

### QR Code Troubleshooting

| Problem | Fix |
|---|---|
| R1 won't read the QR | Make your browser window bigger so the QR is larger on screen |
| Camera can't focus | Adjust distance — try slightly closer or farther away |
| Partial scan failure | Increase screen brightness on your monitor |
| Still not working | Open the QR on your phone and scan from there instead |
| Creation URL error after scan | Double-check your GitHub Pages URL is live and correct in the QR generator |

---

### Updating Your Creation Later

Whenever you change your code, just upload the updated files to your GitHub repo (drag-and-drop or `git push`). GitHub Pages automatically rebuilds and redeploys — still free, no extra steps. Your R1 loads the latest version next time you open the creation.

---

## Method 3 — Install Someone Else's Creation (No Coding, Free)

You do **not** need Intern tasks to install and use creations others have built. This is completely free.

### Option A — Creations Gallery (Browser)
1. Go to **https://rabbit.tech/creations** in a browser
2. Browse community-built creations
3. Click any creation and **scan the QR code** with your R1 to install it

### Option B — Public Tab on Device
1. Open the **Creations card** on your R1
2. Tap the **"Public"** tab
3. Browse and tap **Install** on any creation you want

---

## Pro Tips

- **Be detailed with Intern prompts** — pack all your requirements into one prompt since each one costs a task
- **Use AI coding tools freely** — drop the SDK docs into Claude or ChatGPT and let it write the creation for you, then host and install via QR for free, no Intern credits spent
- **Design for the small screen** — always preview at 240×282 pixels before installing
- **Iterate using source code** — once Intern builds a first version and gives you the source download, you can modify and improve it with the SDK without spending more tasks
- **Share freely** — once hosted, anyone with a QR code can install your creation, and submitting to the gallery at rabbit.tech/creations gets it in front of the whole R1 community
