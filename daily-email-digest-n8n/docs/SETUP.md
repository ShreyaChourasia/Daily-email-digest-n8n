#!/bin/bash
# ============================================
# Git Setup Commands for Daily Email Digest
# Run these in your terminal one by one
# ============================================

# Step 1: Create project folder and navigate into it
mkdir daily-email-digest-n8n
cd daily-email-digest-n8n

# Step 2: Initialize git repository
git init

# Step 3: Create folder structure
mkdir -p workflows docs screenshots

# Step 4: Copy your files into the folders
# (You should have downloaded all files from Claude)
# Place them like this:
#   README.md              → root
#   LICENSE                → root
#   .gitignore             → root
#   daily_email_digest.json → workflows/
#   SETUP.md               → docs/
#   ARCHITECTURE.md        → docs/
#   TROUBLESHOOTING.md     → docs/
#   SCREENSHOT_GUIDE.md    → docs/
#   Your screenshots        → screenshots/

# Step 5: Stage all files
git add .

# Step 6: Create first commit
git commit -m "feat: add daily email digest n8n workflow

- n8n workflow that fetches unread Gmail emails daily at 8 AM
- Summarizes each email using Google Gemini 2.5 Flash
- Posts formatted bullet-point digest to Slack
- Includes error handling for Gmail, Gemini, and empty inbox
- Full documentation: setup guide, architecture, troubleshooting"

# Step 7: Create GitHub repo (using GitHub CLI, or do it manually on github.com)
# Option A: Using GitHub CLI (if installed)
gh repo create daily-email-digest-n8n --public --description "Automated daily email digest using n8n, Gmail, Google Gemini AI, and Slack" --source=. --push

# Option B: Manual (if you don't have GitHub CLI)
# 1. Go to github.com → New Repository
# 2. Name: daily-email-digest-n8n
# 3. Description: Automated daily email digest using n8n, Gmail, Google Gemini AI, and Slack
# 4. Public → Create Repository
# 5. Then run:
git remote add origin https://github.com/YOUR_USERNAME/daily-email-digest-n8n.git
git branch -M main
git push -u origin main

# ============================================
# Done! Your repo is live.
# ============================================