# peek-it Tasker

> Send rich overlay notifications to your Android TV from **Tasker** using **peek-it Send** as a plugin.

peek-it Send acts as a **Tasker Plugin** (Locale API), letting you trigger any peek-it template on your TV from automated tasks — no coding required.

---

## Table of Contents

- [How It Works](#how-it-works)
- [Prerequisites](#prerequisites)
- [Setup](#setup)
- [Creating a Tasker Task](#creating-a-tasker-task)
- [JSON Parameters](#json-parameters)
- [Use Case Examples](#use-case-examples)
- [Troubleshooting](#troubleshooting)
- [API Reference](#api-reference)

---

## How It Works

```
Tasker Event (time, sensor, app, etc.)
  ↓
Tasker fires peek-it Send plugin
  ↓
peek-it Send loads your chosen template from the TV
  ↓
Applies your custom parameters (title, message, etc.)
  ↓
Sends the notification to peek-it TV via HTTP
  ↓
Notification appears as an overlay on TV
```

Your phone and TV just need to be on the **same Wi-Fi network**.

---

## Prerequisites

| What | Details |
|------|---------|
| **peek-it** (TV app) | Installed and running on your Android TV |
| **peek-it Send** (phone app) | Installed on your Android phone |
| **Tasker** | Installed on your Android phone ([Play Store](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm)) |
| **Network** | Phone and TV on the same Wi-Fi / LAN |

---

## Setup

### 1. Configure peek-it Send

Open **peek-it Send** on your phone and configure the TV connection:

1. Enter your **TV IP address** (e.g., `192.168.1.100`)
   - Or tap **Scan TV QR code** to scan it from the TV screen
2. Leave the port as `8081` (default) unless you changed it on the TV
3. *(Optional)* Enter the **API Key** if your TV requires authentication
4. Tap **Save**
5. Tap **Send test notification** to verify the connection

You should see a test notification appear on your TV. If it works, you're ready for Tasker.

### 2. Create Templates on the TV

Tasker sends notifications based on **templates** you've created in peek-it Designer (the web UI on the TV).

- Open a browser and go to `http://<TV_IP>:8081`
- Create templates with placeholder text like `{{title}}` or `{{message}}`
- Save them as **Custom** templates

> Already have templates? Skip this step — Tasker can use any existing template (Official, Custom, or Draft).

---

## Creating a Tasker Task

### Step by Step

1. Open **Tasker**
2. Go to the **Tasks** tab
3. Tap **+** to create a new task, give it a name
4. Tap **+** to add an action
5. Select **Plugin** → **peek-it Send**
6. Tap the **pencil icon** to configure

### Plugin Configuration Screen

| Field | Description | Example |
|-------|-------------|---------|
| **TV** | Shows your configured TV (read-only) | `192.168.1.100:8081` |
| **Template** | Dropdown — pick a template from your TV | `[Custom] Weather Alert` |
| **Duration (ms)** | How long the notification stays on screen | `8000` (8 seconds) |
| **JSON Parameters** | Optional overrides for template placeholders | `{"title": "Hello!"}` |

7. Select a **template** from the dropdown
8. Set the **duration** (default: 8000 ms = 8 seconds)
9. *(Optional)* Add **JSON parameters** to customize the content
10. Tap **Save** (top-right)

### Trigger It

Add a **trigger** to your Tasker profile:
- **Time**: every day at 8:00 AM
- **Event**: phone connected to Wi-Fi
- **App**: when a specific app opens
- **State**: battery below 20%
- Or run the task manually from Tasker's task list

---

## JSON Parameters

JSON parameters let you inject **dynamic content** into your templates at runtime.

### Template Side (on peek-it Designer)

When creating a template, use `{{key}}` placeholders in text elements:

```
Temperature: {{temp}}
Status: {{status}}
```

### Tasker Side (plugin configuration)

In the **JSON Parameters** field, provide matching key-value pairs:

```json
{"temp": "24°C", "status": "Normal"}
```

The TV will replace `{{temp}}` with `24°C` and `{{status}}` with `Normal`.

### Rules

- Must be a valid **JSON object** (curly braces)
- All values must be **strings** (use quotes)
- Keys must match the `{{key}}` placeholders in your template
- Unmatched keys are silently ignored

### Examples

```json
{"title": "Good morning!", "message": "Time to wake up"}
```

```json
{"sensor": "Living Room", "value": "22.5°C", "icon": "mdi:thermometer"}
```

```json
{"camera": "Front Door", "alert": "Motion detected"}
```

> **Tasker variables work too!** Use Tasker's variable syntax in the JSON:
> ```json
> {"time": "%TIME", "battery": "%BATT%", "wifi": "%WIFII"}
> ```

---

## Use Case Examples

### Morning Briefing

| Setting | Value |
|---------|-------|
| Trigger | Time: 07:30 every day |
| Template | `[Custom] Daily Info` |
| Duration | `12000` |
| Params | `{"date": "%DATE", "battery": "%BATT%"}` |

### Low Battery Warning

| Setting | Value |
|---------|-------|
| Trigger | State: Battery Level < 15% |
| Template | `[Custom] Alert` |
| Duration | `8000` |
| Params | `{"title": "Phone Battery Low", "message": "%BATT% remaining"}` |

### Incoming Call on TV

| Setting | Value |
|---------|-------|
| Trigger | Event: Phone Ringing |
| Template | `[Custom] Phone Call` |
| Duration | `15000` |
| Params | `{"caller": "%CNAME", "number": "%CNUM"}` |

### Wi-Fi Connection Notification

| Setting | Value |
|---------|-------|
| Trigger | State: Connected to Home Wi-Fi |
| Template | `[Custom] Welcome Home` |
| Duration | `5000` |
| Params | `{"device": "%HOSTNAME"}` |

### Smart Home Automation (with AutoApps / Join)

| Setting | Value |
|---------|-------|
| Trigger | AutoApps command from Node-RED / MQTT |
| Template | `[Custom] HA Alert` |
| Duration | `10000` |
| Params | `{"entity": "light.living_room", "state": "on"}` |

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| **"TV: not configured"** in plugin screen | Open peek-it Send main app and configure the TV address first |
| **"(TV connection error)"** — no templates load | Check TV IP, verify peek-it is running on TV, check both devices are on same network |
| **"(No templates)"** | Create at least one template in peek-it Designer on the TV |
| **Notification doesn't appear on TV** | 1. Test from peek-it Send main app first. 2. Check API Key matches. 3. Check TV is awake/not in deep sleep |
| **Parameters not replaced** | Verify `{{key}}` syntax in template matches JSON keys exactly (case-sensitive) |
| **Task runs but nothing happens** | Check Tasker logs (long-press task → Test). The plugin has a 5-second timeout — TV must respond in time |
| **API Key error (401/403)** | Enter the correct API Key in peek-it Send. Find it in peek-it Designer → Settings on the TV |

### Quick Diagnostic

1. Open peek-it Send → tap **Send test notification**
   - Works? → TV connection is fine, issue is in Tasker config
   - Fails? → Fix the TV connection first

2. In Tasker, long-press your task → **Test**
   - Check if the notification appears on TV
   - If not, check the template name still exists on TV

---

## API Reference

For advanced usage, direct HTTP integration, payload format, widget types, animations, and more:

**[peek-it API Documentation](https://github.com/jolabs40/peek-it-API)**

The API docs cover:
- Full `POST /api/notify` payload reference
- All 13 widget types (text, image, video, webview, menu, etc.)
- Animation options (fade, slide, pop, etc.)
- Sound and TTS (text-to-speech) configuration
- Template management API
- Home Assistant integration
- And much more

---

## Links

| Resource | URL |
|----------|-----|
| **peek-it API Docs** | [github.com/jolabs40/peek-it-API](https://github.com/jolabs40/peek-it-API) |
| **peek-it (TV app)** | *Available on Google Play (coming soon)* |
| **peek-it Send (phone app)** | *Available on Google Play (coming soon)* |
| **Tasker** | [play.google.com/store/apps/details?id=net.dinglisch.android.taskerm](https://play.google.com/store/apps/details?id=net.dinglisch.android.taskerm) |

---

*peek-it Tasker — Automate your TV notifications*
