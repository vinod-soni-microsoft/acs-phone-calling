# Azure Communication Services Phone Calling

This project demonstrates how to make phone calls using Azure Communication Services Call Automation SDK.

## Prerequisites

1. **Azure Communication Services resource** with phone calling enabled
2. **A phone number purchased** through your ACS resource
3. **Node.js** installed (v18 or higher recommended)

## Important Notes

⚠️ **Phone Number Requirements:**
- `fromPhoneNumber`: Must be a phone number you've purchased through your Azure Communication Services resource
- `toPhoneNumber`: The destination phone number (must be in E.164 format, e.g., +1234567890)
- Format: No dashes, spaces, or parentheses - just `+` and digits

## Setup

1. Install dependencies:
   ```bash
   npm install
   ```

2. **Create a `.env` file** from the example template:
   ```bash
   cp .env.example .env
   ```

3. **Configure your `.env` file** with your Azure credentials:
   ```env
   ACS_CONNECTION_STRING=endpoint=https://your-acs-resource.communication.azure.com/;accesskey=your_access_key
   TO_PHONE_NUMBER=+1234567890
   FROM_PHONE_NUMBER=+1987654321
   CALLBACK_URI=https://your-webhook-url.com/api/callbacks
   ```
   
   - `ACS_CONNECTION_STRING`: Get this from Azure Portal → Communication Services → Keys
   - `FROM_PHONE_NUMBER`: Your ACS phone number (must be purchased through Azure)
   - `TO_PHONE_NUMBER`: Destination phone number (E.164 format)
   - `CALLBACK_URI`: Your public webhook URL (will be generated in Step 1)

## How to Make a Call

### Step 1: Create a Dev Tunnel

First, login to Dev Tunnel (Microsoft's tunneling service):

```bash
devtunnel user login
```

Create a tunnel that allows anonymous access:

```bash
devtunnel create --allow-anonymous
```

Add port 3000 with HTTP protocol:

```bash
devtunnel port create -p 3000 --protocol http
```

Start hosting the tunnel:

```bash
devtunnel host
```

You'll see output like:
```
Hosting port: 3000
Connect via browser: https://abc123.use.devtunnels.ms:3000, https://abc123-3000.use.devtunnels.ms
```

**Copy the HTTPS URL** (e.g., `https://abc123-3000.use.devtunnels.ms`) and update your `.env` file:
```env
CALLBACK_URI=https://abc123-3000.use.devtunnels.ms/api/callbacks
```

### Step 2: Start the Webhook Server

In a **second terminal**, start the webhook server to receive call events:

```bash
npm run webhook
```

This will start a local Express server on port 3000 that listens for Azure call events.

### Step 3: Make the Call

In a **third terminal**, run:

```bash
node dist/index.js
```

Or use the npm script:
```bash
npm run dev
```

This will read your configuration from `.env` and initiate the call using your callback URL.

### Step 4: Monitor Events

Watch the webhook server terminal for real-time call events:
- ✅ `CallConnected` - Call was answered
- 📴 `CallDisconnected` - Call ended
- 👥 `ParticipantsUpdated` - Participants joined/left
- And more...

## Available Scripts

- `npm run webhook` - Start the webhook server on port 3000
- `npm run call` - Make a phone call (interactive, prompts for callback URL)
- `npm run dev` - Run the simple example (uses .env configuration)
- `npm run build` - Compile TypeScript

## Webhook Events

The webhook server handles these Azure Communication Services events:

| Event | Description |
|-------|-------------|
| `CallConnected` | Call successfully connected |
| `CallDisconnected` | Call ended |
| `ParticipantsUpdated` | Participants changed |
| `PlayCompleted` | Audio playback finished |
| `RecordingStateChanged` | Recording started/stopped |
| `RecognizeCompleted` | DTMF/Speech recognition done |

## Troubleshooting

**Missing Environment Variables**
- Ensure `.env` file exists and contains all required variables
- Never commit the `.env` file to git (it's in `.gitignore`)
- Use `.env.example` as a template

**Dev Tunnel Setup**
- Dev Tunnel is included with Visual Studio and VS Code
- Login required: `devtunnel user login`
- Tunnels expire after 30 days (create a new one when needed)
- Alternative: Use ngrok with `ngrok http 3000` if you prefer

**Phone Call Fails**
- Verify your ACS resource has PSTN calling enabled
- Check that the phone number is properly provisioned
- Ensure the callback URL is publicly accessible
- Verify phone numbers are in E.164 format

**Webhook Not Receiving Events**
- Make sure the webhook server is running (`npm run webhook`)
- Verify the dev tunnel is active (`devtunnel host`)
- Ensure the CALLBACK_URI in `.env` matches your dev tunnel URL
- Check firewall settings
- Confirm the tunnel allows anonymous access (`--allow-anonymous`)
