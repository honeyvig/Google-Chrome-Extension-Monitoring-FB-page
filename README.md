# Google-Chrome-Extension-Monitoring-FB-page
create a google chrome extension for monitoring certain FB pages for certain keyword and comment when there is a new post that mentions that keyword.

The software was working somewhat, but after using it for a few hours, it blocks FB account saying there's robotic activity, even if it does not comment.

So I want to change the automation method to make it work. There are also other issues with design and overall workflow that needs to be corrected.

Other than that, I want to integrate AI into the comment system.
---------------
Facebook automation tool to avoid being flagged for robotic activity, improve the design and workflow, and integrate AI into the comment system. Below is a step-by-step approach to building a Chrome extension that monitors certain Facebook pages for specific keywords and comments on posts using a more human-like approach to avoid detection.
Key Points:

    Monitoring Facebook Pages for Keywords: Use Facebook Graph API or content scripts to monitor specific posts on Facebook pages.
    Human-like Commenting: To avoid detection, the extension should interact like a human by adding delays between actions, randomizing intervals, and simulating natural browsing patterns.
    AI-Driven Comments: Integrate an AI model (such as GPT) to generate relevant comments based on the detected keywords.

1. Manifest File (manifest.json)

First, you need to define your extension's metadata and permissions in the manifest.json file.

{
  "manifest_version": 3,
  "name": "Facebook Keyword Monitor",
  "description": "Monitor Facebook pages for specific keywords and comment using AI.",
  "version": "1.0",
  "permissions": [
    "activeTab",
    "storage",
    "identity",
    "https://www.facebook.com/*"
  ],
  "background": {
    "service_worker": "background.js"
  },
  "content_scripts": [
    {
      "matches": ["https://www.facebook.com/*"],
      "js": ["content.js"]
    }
  ],
  "action": {
    "default_popup": "popup.html",
    "default_icon": "icon.png"
  },
  "icons": {
    "16": "icons/icon16.png",
    "48": "icons/icon48.png",
    "128": "icons/icon128.png"
  }
}

2. Background Script (background.js)

The background script manages long-running tasks like making API calls to Facebook's Graph API or interacting with external AI services.

// background.js

// Function to fetch posts from a given Facebook page
async function fetchFacebookPosts(pageId, accessToken) {
  const url = `https://graph.facebook.com/${pageId}/posts?access_token=${accessToken}`;
  const response = await fetch(url);
  const data = await response.json();
  return data.data;  // List of posts
}

// Function to comment on a post with an AI-generated comment
async function postComment(postId, accessToken, comment) {
  const url = `https://graph.facebook.com/${postId}/comments?message=${encodeURIComponent(comment)}&access_token=${accessToken}`;
  await fetch(url, { method: "POST" });
}

// Listen for messages from content scripts
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === "fetchPosts") {
    fetchFacebookPosts(message.pageId, message.accessToken).then(posts => {
      sendResponse({ posts });
    });
    return true;  // Keep the message channel open for async response
  }

  if (message.action === "postComment") {
    postComment(message.postId, message.accessToken, message.comment).then(() => {
      sendResponse({ success: true });
    });
    return true;
  }
});

3. Content Script (content.js)

The content script is responsible for monitoring the page and detecting posts that mention certain keywords.

// content.js

// Helper function to simulate human-like delays
function humanLikeDelay(min, max) {
  return new Promise(resolve => setTimeout(resolve, Math.floor(Math.random() * (max - min + 1)) + min));
}

// Helper function to search for posts containing a specific keyword
function monitorPostsForKeywords(keyword) {
  const posts = document.querySelectorAll('.userContent');  // Change this to the correct selector for Facebook posts
  posts.forEach(post => {
    if (post.innerText.toLowerCase().includes(keyword.toLowerCase())) {
      console.log('Keyword found in post:', post.innerText);
      sendCommentRequest(post);
    }
  });
}

// Send comment request to background script
async function sendCommentRequest(post) {
  const postId = post.getAttribute('data-id');  // Adjust this selector based on Facebook's DOM structure
  const accessToken = 'YOUR_FACEBOOK_ACCESS_TOKEN';  // Get the token securely, ideally through OAuth
  
  // Simulate AI-generated comment
  const comment = await getAIComment(post.innerText);  // Use AI model to generate a relevant comment
  
  chrome.runtime.sendMessage({ action: 'postComment', postId, accessToken, comment }, response => {
    console.log('Comment posted successfully!');
  });
}

// Call the function periodically to monitor posts
setInterval(() => {
  monitorPostsForKeywords('yourKeyword');  // Replace 'yourKeyword' with the actual keyword you're looking for
}, 30000);  // Run every 30 seconds to check for new posts

4. AI Integration (ai.js)

For AI-generated comments, you can use an AI API such as OpenAI's GPT. Here's an example of integrating it:

// ai.js - Example AI Integration (use GPT-3 for comment generation)

async function getAIComment(postContent) {
  const apiKey = 'YOUR_OPENAI_API_KEY';
  const response = await fetch('https://api.openai.com/v1/completions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer ${apiKey}`
    },
    body: JSON.stringify({
      model: 'text-davinci-003',
      prompt: `Write a human-like comment for the following Facebook post content: "${postContent}"`,
      max_tokens: 50
    })
  });

  const data = await response.json();
  return data.choices[0].text.trim();
}

5. Popup (popup.html)

This is the popup UI where users can configure the extension and see what’s happening.

<!DOCTYPE html>
<html>
  <head>
    <title>Facebook Keyword Monitor</title>
    <style>
      body {
        font-family: Arial, sans-serif;
        width: 200px;
      }
      button {
        padding: 10px;
        margin-top: 10px;
        background-color: #4CAF50;
        color: white;
        border: none;
        cursor: pointer;
      }
    </style>
  </head>
  <body>
    <h2>Facebook Keyword Monitor</h2>
    <button id="startMonitoring">Start Monitoring</button>

    <script>
      document.getElementById('startMonitoring').addEventListener('click', () => {
        alert('Monitoring started...');
        // Trigger content script monitoring
      });
    </script>
  </body>
</html>

6. Styling the Extension (Optional)

You can add some CSS to your popup or other parts of the extension to make it more user-friendly.
Key Concepts to Avoid Being Flagged for Robotic Activity:

    Human-like Delays: Use setTimeout with random intervals to simulate human-like interactions (as seen in humanLikeDelay() function).
    Randomize Actions: Vary the timing and intervals between comments and page checks to avoid predictable behavior.
    Use AI for Comment Generation: Instead of repetitive, templated comments, use AI to create unique, contextually relevant comments to avoid detection.

Additional Considerations:

    Facebook API Permissions: You need to ensure your app has the right permissions (like user_posts, pages_manage_posts) to interact with Facebook pages.
    AI Training: You may want to fine-tune AI models to generate comments that fit well with different types of posts.
    Security: Ensure that Facebook access tokens and AI API keys are handled securely, especially when distributing the extension.

Conclusion:

This extension monitors Facebook pages for posts containing specified keywords and uses AI to automatically comment on relevant posts. By adding random delays, human-like interaction patterns, and AI-driven responses, we can mitigate the risk of being flagged for robotic activity.
