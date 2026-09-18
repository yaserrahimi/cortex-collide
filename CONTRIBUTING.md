# 🤝 Contributing to Cortex Collide

Thank you for your interest in contributing to **Cortex Collide**! We welcome bug reports, feature suggestions, documentation enhancements, and new AI model adapters.

---

## 📋 Table of Contents
1. [Development Setup](#-development-setup)
2. [Adding a New AI Chatbot Adapter](#-adding-a-new-ai-chatbot-adapter)
3. [Testing & Quality Assurance](#-testing--quality-assurance)
4. [Translations & Internationalization (i18n)](#-translations--internationalization-i18n)
5. [Pull Request Workflow](#-pull-request-workflow)

---

## 🛠️ Development Setup

1. **Clone the repository:**
   ```bash
   git clone https://github.com/<your-username>/cortex-collide.git
   cd cortex-collide
   ```

2. **Load into Google Chrome:**
   - Open Chrome and navigate to `chrome://extensions/`.
   - Enable **Developer mode** (toggle in the top-right corner).
   - Click **Load unpacked** and select the `cortex-collide` project directory.
   - The extension icon will appear in your Chrome toolbar.

3. **Verify tests:**
   ```bash
   npm test
   npm run check
   ```

---

## 🤖 Adding a New AI Chatbot Adapter

Cortex Collide features a plugin-oriented **Adapter Pattern** that decouples chatbot DOM interactions from the central orchestration logic. Adding support for a new model (e.g., DeepSeek, Grok, Mistral, Perplexity) requires only 3 steps:

### Step 1: Create the Adapter Script
Create a new file in `content/bots/<bot_id>.js` (e.g., `content/bots/deepseek.js`) extending or matching the unified adapter engine:

```javascript
/**
 * Cortex Collide: <Bot Name> Adapter (content/bots/<bot_id>.js)
 */

(function () {
  const adapterConfig = {
    id: 'deepseek',
    name: 'DeepSeek',
    matchesUrl: (url) => url.includes('chat.deepseek.com'),
    findInputField: () => document.querySelector('textarea, div[contenteditable="true"]'),
    findSendButton: () => document.querySelector('button[aria-label="Send"], button.send-btn'),
    isGenerating: () => {
      // Return true if generation / stop button is currently active
      return Boolean(document.querySelector('button[aria-label="Stop"]'));
    },
    extractLatestResponse: () => {
      // Return latest assistant message text from DOM
      const messages = document.querySelectorAll('.message-assistant, [data-message-author="assistant"]');
      if (!messages.length) return '';
      return messages[messages.length - 1].innerText || '';
    }
  };

  if (globalThis.AICD_ADAPTER_ENGINE) {
    globalThis.AICD_ADAPTER_ENGINE.register(adapterConfig);
  }
})();
```

### Step 2: Register in Central Bot Registry
In `background/bots_registry.js`, register the metadata:

```javascript
export const BOTS = {
  // ... existing bots (gemini, chatgpt, claude)
  deepseek: {
    id: 'deepseek',
    name: 'DeepSeek',
    shortName: 'DeepSeek',
    homeUrl: 'https://chat.deepseek.com/',
    deepUrlCheck: (url) => url.includes('/chat/'),
    avatarIcon: '🐋',
    color: '#0ea5e9'
  }
};
```

### Step 3: Update `manifest.json` Permissions
Add the target domain to `host_permissions` and `content_scripts.matches` in `manifest.json`:

```json
"host_permissions": [
  "*://chat.deepseek.com/*"
],
"content_scripts": [
  {
    "matches": [
      "*://chat.deepseek.com/*"
    ],
    "js": [
      "content/core.js",
      "content/adapter.js",
      "content/bots/deepseek.js",
      ...
    ]
  }
]
```

Reload the extension in `chrome://extensions/` and verify connectivity!

---

## 🧪 Testing & Quality Assurance

Before submitting any code changes, ensure all automated tests and syntax checks pass:

```bash
# Run i18n key parity and functional smoke tests
npm test

# Run syntax verification on all JavaScript modules
npm run check

# Verify i18n catalogs specifically
npm run validate:i18n
```

---

## 🌍 Translations & Internationalization (i18n)

Cortex Collide supports **8 languages**: English (`en`), Persian (`fa`), Arabic (`ar`), Spanish (`es`), French (`fr`), German (`de`), Chinese (`zh`), and Russian (`ru`).

- When adding new user-facing strings, add keys to all 8 catalogs in `i18n/translations.js`.
- Always verify with `npm run validate:i18n`. The CI workflow will fail if keys are missing from any language catalog.
- If introducing Chrome extension level strings (store listing, context menus), update all 8 files in `_locales/<lang>/messages.json`. Remember that Chrome requires variables ($var$) to be explicitly defined in `"placeholders"`.

---

## 🚀 Pull Request Workflow

1. Fork the repository and create your feature branch:
   ```bash
   git checkout -b feature/my-new-bot-adapter
   ```
2. Commit your changes with clear, semantic commit messages:
   ```bash
   git commit -m "feat(adapter): add DeepSeek chatbot adapter"
   ```
3. Push to your branch and open a Pull Request.
4. Ensure the GitHub Actions CI workflow passes completely.
