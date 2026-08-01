Title: Building a GoFullPage Clone Using a Chrome Extension
Date: 2026-07-14
Category: Tech
Tags: Chrome Extension, JavaScript, Manifest V3, screenshot, web-development
Slug: building-gofullpage-clone-chrome-extension

Browsers only capture what's visible on screen. GoFullPage solves that by scrolling through the entire page and stitching the captures together. Here's how to build that yourself using Chrome Extension Manifest V3, HTML, CSS, and JavaScript.

## How It Works

The extension automates what you'd otherwise do manually — scroll, screenshot, repeat, stitch. It injects a script into the active tab, scrolls from top to bottom capturing sections, combines them into a single image, and downloads the result automatically.

**The flow:**

```
User clicks icon → popup.html → popup.js → background.js
    → content.js (scrolls page) → fullpage.js (captures + stitches)
    → download image
```

## Project Structure

```
GOFULLPAGE/
├── manifest.json     ← extension config, permissions, entry points
├── popup.html        ← the UI shown when you click the icon
├── popup.js          ← handles the Capture button click
├── background.js     ← central controller, coordinates everything
├── content.js        ← injected into the webpage, scrolls it
├── fullpage.js       ← captures sections, stitches into one image
├── style.css         ← popup styling
└── icons/            ← toolbar and store icons
```

## File Responsibilities

**manifest.json** — The file Chrome reads first. Declares the extension name, version, required permissions, background service worker, popup entry point, and icons. Without it, Chrome won't recognise the project as an extension at all.

**popup.html / popup.js** — The small UI that appears when the user clicks the extension icon. `popup.js` listens for the Capture button click and fires a message to `background.js` to kick off the process.

**background.js** — The central controller. Listens for messages from the popup, manages Chrome Extension APIs, and coordinates between the content script and the screenshot logic. Runs in the background even when the popup is closed.

**content.js** — Injected directly into the active webpage. Reads the full page height, handles the automated scrolling, and sends page dimensions back to the background script. This is the bridge between the extension and the live DOM.

**fullpage.js** — The core feature. Captures each visible section as the page scrolls, then combines all the captured sections into a single long image ready for download.

**style.css** — Controls the popup's visual appearance: button styles, fonts, colors, and spacing. Keeps the interface clean and usable.

## Key Chrome APIs Involved

**chrome.tabs** — Lets the background script query the active tab and inject content scripts into it.

**chrome.scripting** — Manifest V3's way of programmatically injecting `content.js` into the current page at capture time.

**chrome.runtime.sendMessage / onMessage** — The messaging layer that connects popup → background → content script. Each file communicates through structured message passing rather than direct function calls.

**captureVisibleTab** — The Chrome API that takes a screenshot of the currently visible viewport. Called repeatedly as the page scrolls to build up the full-page image.

## The Stitching Logic

Each `captureVisibleTab` call returns a base64-encoded PNG of the visible area. `fullpage.js` draws each capture onto an offscreen `<canvas>` at the correct vertical offset, then exports the entire canvas as the final image. The canvas height equals the full scrollHeight of the page.

## Why Manifest V3?

Manifest V2 used persistent background pages. V3 replaces them with service workers, which are event-driven and don't stay resident. The tradeoff is slightly more complex message coordination, but it's Chrome's current standard and required for new extensions published to the Chrome Web Store.

> Full-page screenshot tools look simple from the outside but touch several non-obvious browser APIs. Building this from scratch gives you a solid foundation in Manifest V3 architecture, cross-context messaging, and the Chrome scripting model.
