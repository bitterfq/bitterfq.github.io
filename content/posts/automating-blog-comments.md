---
title: "Automating Blog Comments with AI: GPT4All & Giscus Integration"
date: 2025-02-03T18:28:04-08:00
draft: false
---

Engagement is crucial for any blog, but manually responding to every post can be time-consuming. What if your blog could generate thoughtful comments automatically? 

I developed a bot that does exactly that—leveraging **GPT4All** for offline AI-generated comments and **Giscus** for seamless integration with GitHub Discussions. This solution eliminates reliance on cloud-based APIs, ensuring full privacy, cost-efficiency, and automation.

<!--more-->

## **🎯 Project Goals**

The objectives of this project were:
1. **Automate blog comment generation** with insightful AI responses.
2. **Utilize a local AI model** instead of cloud-based services like OpenAI.
3. **Integrate with Giscus** to post comments directly into GitHub Discussions.

By automating this process, the system fosters engagement while maintaining full **privacy and control** over the workflow.

---

## **🛠️ Technology Stack**

| Component             | Technology Used        |
|----------------------|----------------------|
| **AI Model**        | GPT4All (Llama 3.2 3B) |
| **Static Site Generator** | Hugo |
| **Commenting System** | Giscus (GitHub Discussions) |
| **Automation**      | Python (Requests, BeautifulSoup) |
| **Hosting**         | GitHub Pages |

---

## **📌 How It Works**

The bot follows a **three-step process**:

1️⃣ **Extracts the latest blog post**  
2️⃣ **Generates an AI-generated comment using GPT4All**  
3️⃣ **Posts the comment to Giscus via GitHub API**  

Each step is designed to be efficient, ensuring the bot can autonomously engage with readers.

---

## **🔍 Step 1: Extracting the Latest Blog Post**

The bot scrapes the blog’s homepage to retrieve the most recent post and extract its content.

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

## **💡 Step 2: Generating AI Comments with GPT4All**

To avoid dependence on cloud-based APIs, I opted for **GPT4All**, a locally hosted AI model, to generate insightful comments.

```python
from gpt4all import GPT4All

def generate_comment(text):
    model_path = "/home/bitterfq/.local/share/nomic.ai/GPT4All/Llama-3.2-3B-Instruct-Q4_0.gguf"
    gpt = GPT4All(model_path)

    prompt = (
        "You are an insightful and thoughtful reader. "
        "Write a meaningful and engaging comment about the following blog post. "
        "Your response should be a self-contained comment without any additional text.\n\n"
        f"Blog Post:\n{text}\n\n"
        "### Comment:\n"
    )

    response = gpt.generate(prompt, max_tokens=200).strip()

    # Append AI signature
    response += "\n\n*This comment is AI-generated.*"

    return response
```

This ensures the comments remain **relevant, concise, and structured**, with a clear indication that they are AI-generated.

---

## **📌 Step 3: Posting the Comment to Giscus**

Once the comment is generated, the bot posts it to the correct GitHub Discussion associated with the blog post.

```python
def post_comment(comment):
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
```

The bot ensures that each comment is correctly associated with its respective post, enabling **seamless discussion automation**.

---

## **🚀 Deployment**

1. **Install dependencies**:
```bash
pip install requests beautifulsoup4 markdownify gpt4all python-dotenv
```

2. **Run the bot**:
```bash
python giscus_bot.py
```

---

## **🎉 Key Takeaways**

✅ **Fully automated** AI-generated blog comments  
✅ **Offline AI processing** with GPT4All—no cloud API costs  
✅ **Seamless GitHub Discussions integration** via Giscus  

This project demonstrates how **automation and AI** can work together to enhance reader engagement while maintaining full control over content generation.

---
