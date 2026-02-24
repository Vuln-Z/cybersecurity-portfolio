# Getting Your Portfolio on GitHub – 5-Minute Guide

This file gives you the exact steps to upload the portfolio to GitHub **right now**.

---

## ✅ Prerequisites

- [ ] GitHub account (github.com)
- [ ] Git installed on your machine (`git --version` should show something)
- [ ] SSH key or personal access token set up (simplest: use GitHub Desktop if you prefer GUI)

---

## 🚀 Method 1: Using GitHub Desktop (Easiest)

1. **Download GitHub Desktop** – https://desktop.github.com
2. **File → Add Local Repository…**
   - Choose: `/home/vulnz/.openclaw/workspace/cybersecurity-portfolio`
   - Name: `cybersecurity-portfolio` (or your GitHub username-cybersecurity)
   - Description: "My hands-on security portfolio: SIEM labs, detection rules, THM write-ups"
   - License: MIT
   - Check "Push to GitHub" – this creates the repo automatically
3. Click **Create Repository**
4. In the next screen, commit with message "Initial portfolio commit" and click **Commit to main**
5. Click **Push origin** (top bar)

Done. Your repo is live at: `https://github.com/yourusername/cybersecurity-portfolio`

---

## 🖥️ Method 2: Command Line (Git)

If you prefer terminal:

```bash
# 1. Navigate to portfolio folder
cd /home/vulnz/.openclaw/workspace/cybersecurity-portfolio

# 2. Initialize git (if not already)
git init

# 3. Add all files
git add .

# 4. Commit
git commit -m "Initial: cybersecurity portfolio with SIEM lab, detection rules, THM notes"

# 5. Create repo on GitHub first (go to github.com → New → name it cybersecurity-portfolio)
#    Do NOT initialize with README (you already have one)

# 6. Link local to remote (replace YOURUSERNAME)
git remote add origin https://github.com/YOURUSERNAME/cybersecurity-portfolio.git

# 7. Push
git branch -M main
git push -u origin main
```

---

## 🔧 After Upload: Enable GitHub Pages (Optional)

Want a nice portfolio website? Enable GitHub Pages:

1. Go to repo → Settings → Pages
2. Source: `Deploy from a branch`
3. Branch: `main`, folder: `/root` (or `/docs`)
4. Save. Site will be: `https://yourusername.github.io/cybersecurity-portfolio`

If you want a custom domain, you can add it there too.

---

## ✨ Final Checklist

- [ ] Repository is **public** (not private)
- [ ] All files uploaded (no missing markdown)
- [ ] In `README.md`, replace placeholder:
  - `[Your Email]` → your actual email
  - `[City, State]` → your location
  - `[github.com/yourusername]` → your profile link
- [ ] Optionally add a profile picture (in GitHub UI)
- [ ] Pin the repo to your profile (repo → Settings → Pin to profile)

---

## 📌 Use Your Portfolio

Once live:

1. **Update your resume** – add link: `https://github.com/yourusername/cybersecurity-portfolio`
2. **Add to LinkedIn** – in "Featured" section or Licenses & Certifications
3. **Mention in interviews** – "You can see my detection rules on GitHub"
4. **Keep updating** – add new THM rooms, new detection rules, new labs

---

## ❓ Troubleshooting

**"Git not found"** → Install Git: `sudo apt install git` (Ubuntu/Debian)

**"Permission denied (publickey)"** → Set up SSH key or use HTTPS with token:
```bash
git remote set-url origin https://github.com/YOURUSERNAME/cybersecurity-portfolio.git
```
Then you'll be prompted for username/password (use a Personal Access Token as password).

**"Repository not found"** → Did you create the repo on GitHub *before* pushing? Create empty repo first.

---

**Questions?** I'm here to help. Once it's live, let me know the URL and I'll review it with you.
