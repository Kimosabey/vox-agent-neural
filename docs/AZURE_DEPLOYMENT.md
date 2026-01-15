# ☁️ Azure Deployment Guide

**Target Service:** Azure Web App for Containers (Linux)  
**Architecture:** Docker (Single Container)  
**Project:** VoxAgent Neural

---

## 1. Prerequisites

- [Azure CLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli) installed
- Docker installed locally
- Active Azure Subscription

---

## 2. Prepare the Application

We use a **Multi-Stage Dockerfile** to build both Frontend and Backend into one image:

1. **Builds React App**: `npm run build` → `/app/frontend/dist`
2. **Setup Python**: Installs FastAPI, Whisper, FFmpeg
3. **Merge**: Copies React build to `/app/static`
4. **Result**: FastAPI serves both API and React UI

---

## 3. Deployment Steps

### Step 1: Login
```bash
az login
```

### Step 2: Create Resource Group
```bash
az group create --name VoxAgentResourceGroup --location eastus
```

### Step 3: Create Container Registry (ACR)
```bash
az acr create --resource-group VoxAgentResourceGroup --name voxagentregistry --sku Basic --admin-enabled true
```

### Step 4: Build & Push Image
```bash
# Login to ACR
az acr login --name voxagentregistry

# Build and Push (This runs the Dockerfile)
az acr build --registry voxagentregistry --image voxagent-app:v1 .
```
*(This may take 5-10 mins as it installs dependencies)*

### Step 5: Create App Service Plan
```bash
# create a B1 plan (Basic Linux) - good starting point
az appservice plan create --name VoxAgentPlan --resource-group VoxAgentResourceGroup --sku B1 --is-linux
```

### Step 6: Create Web App
```bash
az webapp create --resource-group VoxAgentResourceGroup --plan VoxAgentPlan --name voxagent-speech-app --deployment-container-image-name voxagentregistry.azurecr.io/voxagent-app:v1
```

### Step 7: Configure Variables
```bash
# Set environment variables
az webapp config appsettings set --resource-group VoxAgentResourceGroup --name voxagent-speech-app --settings \
  LIVEKIT_URL="your_url" \
  LIVEKIT_API_KEY="your_key" \
  LIVEKIT_API_SECRET="your_secret" \
  WEBSITES_PORT=8000
```

---

## 4. Verification

1. Go to: `https://voxagent-speech-app.azurewebsites.net`
2. You should see the React App!
3. Test recording (WebSocket will connect automatically)

---

## 5. Troubleshooting

**Logs:**
```bash
az webapp log tail --name voxagent-speech-app --resource-group VoxAgentResourceGroup
```

**Common Issues:**
- `Port 8000`: Ensure `WEBSITES_PORT=8000` is set.
- `Startup Time`: Whisper takes a few seconds to load. Set "Always On" in Azure configurations.
- `Memory`: If it crashes, upgrade plan to B2 or P1V2 (Whisper typically needs ~1GB RAM).

---

## 6. Continuous Deployment (CI/CD)

To auto-deploy when you push to Git:

1. Go to **Deployment Center** in Azure Portal
2. Select **GitHub** as source
3. Select your repo & branch
4. Azure will create a GitHub Action for you automatically!
