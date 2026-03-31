# 🔗 Deep Linking & Short Link System (Flutter + FastAPI)

Production-ready guide for implementing **Android App Links + iOS Universal Links + Backend Short Links**

---

## 📌 Overview

This document defines the complete architecture for:

* Deep Linking (open specific screen inside app)
* Deferred Deep Linking (install → open flow)
* Backend Short Links (shareable URLs for WhatsApp, etc.)

> ⚠️ Firebase Dynamic Links is deprecated. Do NOT use it.

---

# 🏗️ Architecture
```
User clicks link (WhatsApp)
→ https://yourdomain.com/s/abc123
→ Backend resolves
→ Redirects to https://yourdomain.com/chat?programId=123
→ OS intercepts (App Links / Universal Links)
→ App opens directly (no browser)
→ Flutter handles navigation
```

---

# 🧠 How App Opens (Important)

### ✅ Ideal Case (No Browser)
```
User taps link
→ OS checks domain ownership
→ App is verified
→ App opens directly ✅
```

### ❗ Fallback Case (Browser Opens First)
```
User taps link
→ Opens browser (WebView)
→ Redirects to app
```

### Why this happens:

* Domain not verified
* App not installed
* App like WhatsApp forces webview

👉 Your system must support BOTH flows.

---

# 🌐 Deep Link Structure

All links must follow:
```
https://yourdomain.com/{path}?params
```

---

## 📍 Supported Routes

| Feature       | Path           | Params          |
| ------------- | -------------- | --------------- |
| Home          | /home          | -               |
| Chat          | /chat          | programId       |
| Chat List     | /chats         | -               |
| Certificates  | /certificates  | -               |
| Leaderboard   | /leaderboard   | -               |
| Goals         | /goals         | goalId          |
| Habits        | /habits        | habitId         |
| Daily Content | /daily-content | date (optional) |

---

## 🔗 Example Links
```
https://yourdomain.com/chat?programId=123
https://yourdomain.com/goals?goalId=456
https://yourdomain.com/daily-content
```

---

# ⚙️ Backend Responsibilities (FastAPI)

## 1. Short Link Generation

### Endpoint
```
POST /short-link
```

### Request
```json
{
  "deep_link": "https://yourdomain.com/chat?programId=123"
}
```

### Response
```json
{
  "short_url": "https://yourdomain.com/s/abc123"
}
```

---

## 2. Redirect Endpoint

### Endpoint
```
GET /s/{code}
```

### Flow

1. Fetch mapping from DB
2. Redirect to actual deep link

---

## 3. Optional Resolve API
```
GET /deeplink/{code}
```

---

## 🗄️ Database Schema
```json
{
  "code": "abc123",
  "deep_link": "https://yourdomain.com/chat?programId=123",
  "created_at": "timestamp",
  "expiry": null
}
```

---

# 🌐 Domain Configuration (CRITICAL)

## 🤖 Android (App Links)

### File:
```
https://yourdomain.com/.well-known/assetlinks.json
```

### Hosted by:

👉 Backend (FastAPI or server)

### Example:
```json
[
  {
    "relation": ["delegate_permission/common.handle_all_urls"],
    "target": {
      "namespace": "android_app",
      "package_name": "com.example.app",
      "sha256_cert_fingerprints": ["YOUR_SHA256"]
    }
  }
]
```

---

## 🍎 iOS (Universal Links)

### File:
```
https://yourdomain.com/.well-known/apple-app-site-association
```

### Hosted by:

👉 Backend

---

## ⚠️ Requirements

* HTTPS only
* Publicly accessible
* No redirects
* Correct content-type

---

# 📱 Flutter Implementation

## 1. Install package
```yaml
app_links: ^6.0.0
```

---

## 2. Listen for links
```dart
final appLinks = AppLinks();

appLinks.uriLinkStream.listen((uri) {
  if (uri != null) handleDeepLink(uri);
});
```

---

## 3. Cold start
```dart
final uri = await appLinks.getInitialAppLink();
if (uri != null) handleDeepLink(uri);
```

---

## 4. Routing
```dart
void handleDeepLink(Uri uri) {
  switch (uri.path) {
    case '/home': goToHome(); break;
    case '/chat': goToChat(uri.queryParameters['programId']); break;
    case '/chats': goToChatList(); break;
    case '/certificates': goToCertificates(); break;
    case '/leaderboard': goToLeaderboard(); break;
    case '/goals': goToGoals(uri.queryParameters['goalId']); break;
    case '/habits': goToHabits(uri.queryParameters['habitId']); break;
    case '/daily-content': goToDailyContent(uri.queryParameters['date']); break;
    default: goToHome();
  }
}
```

---

# 🔐 Edge Cases

## User not logged in

* Save link
* Redirect to login
* Resume after login

## Invalid link

* Redirect to home

## Expired link

* Backend handles fallback

---

# 🔄 Deferred Deep Linking

## Option 1 (Custom)

* Store pending link in backend
* Fetch after login

## Option 2

* Use Branch.io

---

# 📊 Enhancements

* Analytics tracking
* Campaign tracking
* UTM parameters

---

# ✅ Final Checklist

### Backend

* [ ] Short link API
* [ ] Redirect endpoint
* [ ] assetlinks.json hosted
* [ ] apple-app-site-association hosted

### Mobile

* [ ] App Links configured
* [ ] Deep link parsing
* [ ] Navigation implemented

---

# 🚀 Summary

* Use App Links + Universal Links
* Backend hosts verification files
* Backend manages short links
* Flutter handles navigation

---

**This document is the contract between backend and mobile teams.**
