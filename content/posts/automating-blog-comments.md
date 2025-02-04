---
title: 'Automating Blog Comments with AI: GPT4All & Giscus Integration'
date: '2025-02-03T18:28:04-08:00'
draft: false
tags: ['AI', 'GPT4All', 'Hugo', 'Giscus', 'Automation']
---

What if your blog could automatically generate thoughtful comments for every post? I built a bot that does just that—using **GPT4All for offline AI** and **Giscus for seamless GitHub integration**. No cloud APIs, no recurring costs—just smart automation that engages with your blog posts.  

<!--more-->

## **🔥 Introduction**  

I recently built an **AI-powered comment bot** for my Hugo blog that:  
✅ **Generates meaningful comments** on blog posts.  
✅ **Uses GPT4All** for **fully offline AI generation** (no OpenAI API fees).  
✅ **Posts comments to GitHub Discussions via Giscus** for seamless integration.  

This project automates reader engagement and creates an interactive discussion experience **without manual effort**. Here’s how it works.  

---


## **🎯 The Goal**  

The main objectives were:  
1. **Generate insightful comments** automatically for blog posts.  
2. **Use a local AI model** instead of relying on cloud-based APIs.  
3. **Integrate seamlessly with Giscus**, the GitHub-based comment system for Hugo.  

After some trial and error, I ended up using **GPT4All with Llama 3.2 3B** for AI-powered responses and **GitHub's GraphQL API** for comment posting.

---

## **🛠️ Tech Stack**  

| Component  | Technology Used |
|------------|----------------|
| **AI Model** | GPT4All (Llama 3.2 3B) |
| **Static Site Generator** | Hugo |
| **Comments System** | Giscus (GitHub Discussions) |
| **Automation** | Python (Requests, BeautifulSoup) |
| **Hosting** | GitHub Pages |

---

## **💡 How It Works**  

The bot follows a **three-step process**:  

1️⃣ **Extracts the latest blog post**  
2️⃣ **Generates a comment using GPT4All**  
3️⃣ **Posts the comment to Giscus via GitHub API**  

Let’s break it down.

---

## **📌 Step 1: Extract the Latest Blog Post**  

We first **scrape the latest blog post** from Hugo’s index page.  
The bot looks for the most recent post URL and extracts its content.

```python
import requests
from bs4 import BeautifulSoup
import markdownify

# Fetch latest post
BLOG_URL = "https://bitterfq.github.io/"
response = requests.get(BLOG_URL)
soup = BeautifulSoup(response.text, "html.parser")

latest_post = soup.select_one(".list-container .post-line .line-title a")
if not latest_post:
    print("❌ No posts found.")
    exit(1)

latest_post_link = latest_post["href"]
post_url = f"https://bitterfq.github.io{latest_post_link}"

# Extract blog content
post_response = requests.get(post_url)
post_soup = BeautifulSoup(post_response.text, "html.parser")
post_content_div = post_soup.select_one(".single-content")

post_content = post_content_div.get_text()
post_markdown = markdownify.markdownify(post_content)
```

---

## **📌 Step 2: Generate a Comment Using GPT4All**  

Instead of OpenAI’s API, we use **GPT4All with Llama 3.2 3B**, a **free and offline** model.

```python
from gpt4all import GPT4All

def generate_comment(text):
    model_path = "/home/bitterfq/.local/share/nomic.ai/GPT4All/Llama-3.2-3B-Instruct-Q4_0.gguf"
    gpt = GPT4All(model_path)

    prompt = (
        "You are an insightful reader. "
        "Write a meaningful comment about the following blog post without extra text.\n\n"
        f"Blog Post:\n{text}\n\n"
        "### Comment:\n"
    )

    response = gpt.generate(prompt, max_tokens=200).strip()
    
    # Append AI signature
    response += "\n\n*This comment is AI generated.*"

    return response
```

This ensures the comment **stays relevant** and **doesn’t include unnecessary boilerplate**.

---

## **📌 Step 3: Post the Comment to Giscus**  

We **retrieve the correct GitHub Discussion ID** and post the AI-generated comment.

### **Retrieve the Global Discussion ID**
```python
def get_latest_discussion_id():
    url = "https://api.github.com/graphql"
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github.v3+json"
    }

    query = """
    query {
      repository(owner: "bitterfq", name: "bitterfq.github.io") {
        discussions(first: 5, orderBy: {field: CREATED_AT, direction: DESC}) {
          nodes {
            id
            title
          }
        }
      }
    }
    """

    response = requests.post(url, json={"query": query}, headers=headers)
    discussions = response.json().get("data", {}).get("repository", {}).get("discussions", {}).get("nodes", [])
    
    return discussions[0]["id"] if discussions else None
```

### **Post the Comment**
```python
def post_comment(comment):
    discussion_id = get_latest_discussion_id()
    if not discussion_id:
        print("❌ No valid discussion ID found.")
        return None

    url = "https://api.github.com/graphql"
    headers = {
        "Authorization": f"token {GITHUB_TOKEN}",
        "Accept": "application/vnd.github.v3+json"
    }

    mutation = """
    mutation($discussionId: ID!, $body: String!) {
        addDiscussionComment(input: {discussionId: $discussionId, body: $body}) {
            comment {
                id
                url
            }
        }
    }
    """

    variables = {"discussionId": discussion_id, "body": comment}
    response = requests.post(url, json={"query": mutation, "variables": variables}, headers=headers)

    print(f"🔍 Full API Response: {response.json()}")
    return response.json()
```

---

## **🛠️ Deployment & Running the Bot**  

1️⃣ **Ensure dependencies are installed:**
```bash
pip install requests beautifulsoup4 markdownify gpt4all python-dotenv
```
2️⃣ **Run the bot:**  
```bash
python giscus_bot.py
```
✔ The AI bot **automatically posts comments** to my blog!

---

## **💡 Key Takeaways**  

🚀 **Fully Automated** → The bot handles everything from **reading, generating, and posting comments**.  
💰 **Free & Offline** → GPT4All replaces expensive cloud-based AI APIs.  
⚡ **Seamless GitHub Integration** → Giscus allows easy, **open-source discussions**.  

This project demonstrates how **automation + AI** can **enhance reader engagement** without manual effort.  

---


