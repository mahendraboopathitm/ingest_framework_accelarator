# Google Connectors Setup Documentation

**Date:** July 17, 2026  
**Project:** Databricks Ingest Framework  
**Status:** In Progress - Connector definitions created, awaiting credential configuration

---

## Table of Contents

1. [Overview](#overview)
2. [What We Created](#what-we-created)
3. [Google Connectors Architecture](#google-connectors-architecture)
4. [Connector Details](#connector-details)
5. [Setup Instructions](#setup-instructions)
6. [OAuth Flow Explanation](#oauth-flow-explanation)
7. [Credentials Checklist](#credentials-checklist)
8. [Deployment Configuration](#deployment-configuration)
9. [Next Steps](#next-steps)

---

## Overview

We are implementing **three Google-based connectors** for the Databricks Ingest Framework to enable data ingestion from Google services:

| Connector | Purpose | Status | Complexity |
|---|---|---|---|
| **Google Drive** | Ingest files from Google Drive | Created | Low |
| **Google Analytics 4 (GA4)** | Ingest analytics data | Created | Medium |
| **Google Ads** | Ingest advertising data | Created | High |

These connectors follow the existing framework pattern (like YouTube and HubSpot) and use the same deployment mechanism via YAML configuration.

---

## What We Created

### Directory Structure

```
connectors/
├── google_drive/
│   ├── connector.yaml      # Metadata definition
│   ├── README.md           # Setup instructions
│   └── example.yaml        # Example deployment config
├── google_analytics4/
│   ├── connector.yaml      # Metadata definition
│   ├── README.md           # Setup instructions
│   └── example.yaml        # Example deployment config
├── google_ads/
│   ├── connector.yaml      # Metadata definition
│   ├── README.md           # Setup instructions
│   └── example.yaml        # Example deployment config
├── hubspot/                # Existing
└── youtube/                # Existing
```

### Files Created

#### 1. **Connector Definition Files** (`connector.yaml`)
Each connector has a metadata file that defines:
- Connector name and version
- Description
- Required OAuth scopes
- Credential requirements
- Available data tables

**Example structure:**
```yaml
name: google_analytics4
version: 1.0.0
description: "Ingest Google Analytics 4 data..."

required_config:
  secret_scope: "Databricks secret scope"
  oauth_scopes:
    - "https://www.googleapis.com/auth/analytics.readonly"

credentials:
  client_id: "OAuth Client ID"
  client_secret: "OAuth Client Secret"
  refresh_token: "OAuth Refresh Token"
  property_id: "GA4 Property ID"

available_tables:
  - name: campaign_performance
    description: "Campaign metrics..."
  - name: user_acquisition
    description: "User acquisition data..."
```

#### 2. **Documentation Files** (`README.md`)
Each connector has detailed setup instructions including:
- Configuration requirements
- Step-by-step setup guide
- Credential collection instructions
- Example deployment configuration
- Available tables description

#### 3. **Example Deployment Configs** (`example.yaml`)
Sample deployment YAML files showing how to configure each connector for a customer deployment.

---

## Google Connectors Architecture

### Data Flow

```
┌─────────────────────────────────────────────────────────────────┐
│                     Google Cloud Console                         │
│  (OAuth Setup, API Keys, Credentials Management)                │
└────────────────────────────┬────────────────────────────────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
        ┌───────────▼──────────┐  ┌──▼────────────────┐
        │  Create OAuth        │  │  Create Service   │
        │  Credentials         │  │  Account (Drive)  │
        └───────────┬──────────┘  └──┬─────────────────┘
                    │                 │
        ┌───────────▼─────────────────▼──────────┐
        │  Credentials stored in Databricks      │
        │  Secret Scope: customer_a_secrets      │
        └───────────┬──────────────────────────┘
                    │
        ┌───────────▼──────────────────────┐
        │  Deployment Configuration        │
        │  (deployment.yaml)               │
        └───────────┬──────────────────────┘
                    │
        ┌───────────▼──────────────────────┐
        │  Databricks Deploy Script        │
        │  (scripts/deploy.py)             │
        └───────────┬──────────────────────┘
                    │
        ┌───────────▼──────────────────────┐
        │  Ingestion Notebook Generated    │
        │  (Auto-created in workspace)     │
        └───────────┬──────────────────────┘
                    │
        ┌───────────▼──────────────────────┐
        │  Delta Live Tables Pipeline      │
        │  (Scheduled ingestion)           │
        └───────────┬──────────────────────┘
                    │
        ┌───────────▼──────────────────────┐
        │  Data in Unity Catalog           │
        │  (ingest_framework.bronze)       │
        └──────────────────────────────────┘
```

---

## Connector Details

### 1. Google Drive Connector

**Purpose:** Ingest unstructured and structured files from Google Drive

**Credential Type:** Service Account (No OAuth needed)

**Required Credentials:**
- Service Account JSON key
- Google Drive folder ID

**Available Tables:**
- `file_metadata` - File names, sizes, modification dates, MIME types
- `file_contents` - File contents (CSV, JSON, Parquet supported)
- `folder_structure` - Hierarchical folder structure

**Setup Complexity:** ⭐ Low (Simple file sharing)

**Key Setup Steps:**
1. Create service account in Google Cloud
2. Download JSON key
3. Share target folder with service account email
4. Get folder ID from Google Drive URL
5. Store credentials in Databricks

---

### 2. Google Analytics 4 (GA4) Connector

**Purpose:** Ingest analytics data for campaign performance and customer journey analysis

**Credential Type:** OAuth 2.0 (User-based authentication)

**Required Credentials:**
- OAuth Client ID
- OAuth Client Secret
- OAuth Refresh Token (auto-generated during first auth)
- GA4 Property ID

**Available Tables:**
- `campaign_performance` - Campaign metrics (impressions, clicks, conversions)
- `user_acquisition` - User acquisition channels
- `event_data` - Raw event-level data
- `conversion_funnel` - Multi-step conversion analysis
- `user_demographics` - User demographic insights

**Setup Complexity:** ⭐⭐ Medium (OAuth flow required)

**Key Setup Steps:**
1. Enable Google Analytics Data API
2. Create OAuth 2.0 credentials
3. Get GA4 Property ID
4. During deployment, complete OAuth authentication in browser
5. Refresh token auto-captured

---

### 3. Google Ads Connector

**Purpose:** Ingest advertising data for campaign analysis and performance tracking

**Credential Type:** OAuth 2.0 + Developer Token

**Required Credentials:**
- OAuth Client ID
- OAuth Client Secret
- OAuth Refresh Token (auto-generated during first auth)
- Google Ads Developer Token
- Google Ads Customer ID

**Available Tables:**
- `campaigns` - Campaign data (status, budget, bidding strategy)
- `ad_groups` - Ad group data with performance metrics
- `ads` - Individual ad data
- `keywords` - Keyword data (match types, bids, quality scores)
- `performance_metrics` - Aggregated metrics (impressions, clicks, cost, conversions)
- `search_terms` - Search terms that triggered ads

**Setup Complexity:** ⭐⭐⭐ High (Developer token approval required, takes 24 hours)

**Key Setup Steps:**
1. Enable Google Ads API
2. Request developer token (24 hour approval)
3. Create OAuth 2.0 credentials
4. Get Customer ID
5. During deployment, complete OAuth authentication
6. Refresh token auto-captured

---

## Setup Instructions

### Phase 1: Google Cloud Setup (Already Completed ✅)

#### Created OAuth Credentials

**Date:** July 17, 2026

**Credentials Created:**
- ✅ OAuth 2.0 Application: "Databricks Google Connectors"
- ✅ Application Type: Web Application
- ✅ Authorized Redirect URI: `http://localhost:8888/callback`
- ✅ Authorized JavaScript Origin: `http://localhost:8888`

**Credentials Stored in Databricks (customer_a_secrets scope):**

This single OAuth app is shared by **both GA4 and Google Ads connectors** because they use different scopes to request different permissions.

---

### Phase 2: Per-Connector Setup (In Progress)

#### Google Drive Setup

**Step 1: Obtain Service Account JSON**
```
Location: Google Cloud Console → APIs & Services → Credentials
Action: 
  - Find or create a Service Account
  - Generate JSON key
  - Download the JSON file
```

**Step 2: Share Google Drive Folder**
```
Location: Google Drive
Action:
  - Open the target folder
  - Share with service account email: your-service-account@project.iam.gserviceaccount.com
  - Note the Folder ID from URL: https://drive.google.com/drive/folders/{FOLDER_ID}
```

**Step 3: Store in Databricks**
```bash
databricks secrets put-secret customer_a_secrets google_drive_service_account --string-value "$(cat path/to/service-account.json)"
databricks secrets put-secret customer_a_secrets google_drive_folder_id --string-value "YOUR_FOLDER_ID"
```

---

#### Google Analytics 4 Setup

**Step 1: Get GA4 Property ID**
```
Location: Google Analytics (analytics.google.com)
Action:
  - Click Admin (bottom left)
  - Select your property
  - Property Settings → Property ID (10-digit number)
  - Save this value
```

**Step 2: Store in Databricks**
```bash
databricks secrets put-secret customer_a_secrets google_ga4_property_id --string-value "YOUR_PROPERTY_ID"
databricks secrets put-secret customer_a_secrets google_ga4_refresh_token --string-value "WILL_AUTO_POPULATE"
```

**Step 3: Refresh Token Auto-Capture**
- During first deployment, Databricks notebook opens browser
- User authenticates with Google
- Refresh token automatically captured and stored

---

#### Google Ads Setup

**Step 1: Request Developer Token** ⚠️ Takes 24 hours
```
Location: Google Ads (ads.google.com) - MCC Account
Action:
  - Click Tools & Settings → API Center
  - Click "Get access to Google Ads API"
  - Complete form and submit
  - Wait for approval (typically 24 hours)
  - Copy Developer Token when approved
```

**Step 2: Get Customer ID**
```
Location: Google Ads
Action:
  - Top-right corner shows account number: 123-456-7890
  - Save this value
```

**Step 3: Store in Databricks**
```bash
databricks secrets put-secret customer_a_secrets google_ads_developer_token --string-value "YOUR_DEVELOPER_TOKEN"
databricks secrets put-secret customer_a_secrets google_ads_customer_id --string-value "123-456-7890"
databricks secrets put-secret customer_a_secrets google_ads_refresh_token --string-value "WILL_AUTO_POPULATE"
```

**Step 4: Refresh Token Auto-Capture**
- During first deployment, Databricks notebook opens browser
- User authenticates with Google
- Refresh token automatically captured and stored

---

## OAuth Flow Explanation

### Why Different Scopes Matter

**Single OAuth App, Multiple Services:**
```
Client ID: 9315373071-j05ndbge0irbr9os4uq1fl1vd12679du.apps.googleusercontent.com
Client Secret: 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
```

**Different Scopes = Different Permissions:**

```
┌─────────────────────────────────────────────────────────────┐
│              When GA4 Connector Authenticates                │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Scope Requested: https://www.googleapis.com/auth/analytics  │
│  User Sees: "Databricks wants to view your Analytics data"  │
│  Permission: Read-only access to Google Analytics           │
│  Result: Refresh token for Analytics (can read GA4 data)    │
│                                                               │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│              When Google Ads Connector Authenticates          │
├─────────────────────────────────────────────────────────────┤
│                                                               │
│  Scope Requested: https://www.googleapis.com/auth/adwords   │
│  User Sees: "Databricks wants to access your Ads account"  │
│  Permission: Access to Google Ads API                       │
│  Result: Refresh token for Ads (can read Ads data)          │
│                                                               │
└─────────────────────────────────────────────────────────────┘
```

### Scopes Used in Project

| Connector | Scope | Purpose |
|---|---|---|
| Google Drive | `https://www.googleapis.com/auth/drive.readonly` | Read-only access to Drive files |
| GA4 | `https://www.googleapis.com/auth/analytics.readonly` | Read-only access to Analytics data |
| Google Ads | `https://www.googleapis.com/auth/adwords` | Access to Ads API |

---

## Credentials Checklist

### Phase 1: General OAuth Setup ✅ COMPLETE

- ✅ Google Cloud Project created/selected
- ✅ APIs enabled (Drive, Analytics, Ads)
- ✅ OAuth 2.0 App created: "Databricks Google Connectors"
- ✅ Client ID stored in Databricks
- ✅ Client Secret stored in Databricks

### Phase 2: Google Drive Setup ⏳ NOT STARTED

- ⏳ Service Account created
- ⏳ Service Account JSON downloaded
- ⏳ JSON key stored in Databricks
- ⏳ Drive folder shared with service account
- ⏳ Folder ID stored in Databricks

### Phase 3: GA4 Setup ⏳ PARTIALLY STARTED

- ✅ Client ID & Secret stored
- ⏳ GA4 Property ID obtained and stored
- ⏳ First-time authentication completed (browser flow)
- ⏳ Refresh token auto-captured and stored

### Phase 4: Google Ads Setup ⏳ NOT STARTED

- ✅ Client ID & Secret stored
- ⏳ Developer Token requested
- ⏳ Developer Token approved and stored
- ⏳ Customer ID obtained and stored
- ⏳ First-time authentication completed (browser flow)
- ⏳ Refresh token auto-captured and stored

---

## Deployment Configuration

### Current Deployment File

**Location:** `deployments/customer_a/deployment.yaml`

**Current State (YouTube only):**
```yaml
customer:
  name: "customer_a"
  workspace_url: "https://dbc-ce0ce842-10b5.cloud.databricks.com"
  catalog: "ingest_framework"
  schema: "bronze"
  secret_scope: "customer_a_secrets"

connectors:
  - name: "youtube"
    connection_name: "youtube_api_connection"
    credentials:
      api_key: "youtube_api_key"
    tables:
      - source_table: "search"
        destination_table: "search_raw"
        table_configuration:
          q: "databricks"
      - source_table: "activities"
        destination_table: "activities_raw"
        table_configuration:
          channel_id: "UCxOVEOhPNXcJuyfVLhm_BfQ"
```

### How to Add Google Connectors

#### To Add Google Drive:
```yaml
  - name: "google_drive"
    connection_name: "customer_a_google_drive"
    credentials:
      service_account_json: "google_drive_service_account"
      folder_id: "google_drive_folder_id"
    tables:
      - source_table: "file_metadata"
        destination_table: "google_drive_file_metadata"
      - source_table: "file_contents"
        destination_table: "google_drive_files"
```

#### To Add GA4:
```yaml
  - name: "google_analytics4"
    connection_name: "customer_a_ga4"
    credentials:
      client_id: "google_client_id"
      client_secret: "google_client_secret"
      refresh_token: "google_ga4_refresh_token"
      property_id: "google_ga4_property_id"
    tables:
      - source_table: "campaign_performance"
        destination_table: "ga4_campaign_performance"
      - source_table: "user_acquisition"
        destination_table: "ga4_user_acquisition"
```

#### To Add Google Ads:
```yaml
  - name: "google_ads"
    connection_name: "customer_a_google_ads"
    credentials:
      client_id: "google_client_id"
      client_secret: "google_client_secret"
      refresh_token: "google_ads_refresh_token"
      developer_token: "google_ads_developer_token"
      customer_id: "google_ads_customer_id"
    tables:
      - source_table: "campaigns"
        destination_table: "google_ads_campaigns"
      - source_table: "performance_metrics"
        destination_table: "google_ads_performance"
```

---

## Next Steps

### Immediate Actions

1. **Choose Priority Connector**
   - Recommend: Google Analytics 4 (simplest)
   - Alternative: Google Drive (no OAuth complexity)
   - Complex: Google Ads (24-hour developer token approval)

2. **Collect Credentials**
   - Follow per-connector setup instructions above
   - Store each credential in Databricks secrets

3. **Update Deployment YAML**
   - Add connector config to `deployments/customer_a/deployment.yaml`
   - Run validation: `python scripts/validate.py deployments/customer_a/deployment.yaml`

4. **Run Deployment**
   - Execute: `python scripts/deploy.py deployments/customer_a/deployment.yaml`
   - For OAuth connectors: authenticate in browser when prompted
   - Monitor logs for successful ingestion

### Testing Strategy

1. **Single Connector First**
   - Test one connector at a time
   - Verify data appears in Unity Catalog

2. **Multi-Connector Deployment**
   - Once one works, add additional connectors
   - Can run with YouTube + Google Drive + GA4 + Google Ads

3. **Production Rollout**
   - Create separate deployments for different customers
   - Each customer gets own secret scope
   - Each customer can choose which connectors to use

---

## Summary of Changes Made

### Files Created

```
✅ connectors/google_drive/connector.yaml
✅ connectors/google_drive/README.md
✅ connectors/google_drive/example.yaml

✅ connectors/google_analytics4/connector.yaml
✅ connectors/google_analytics4/README.md
✅ connectors/google_analytics4/example.yaml

✅ connectors/google_ads/connector.yaml
✅ connectors/google_ads/README.md
✅ connectors/google_ads/example.yaml

✅ GOOGLE_CONNECTORS_SETUP.md (this file)
```

### Credentials Stored in Databricks

```
Scope: customer_a_secrets

✅ google_client_id:     9315373071-j05ndbge0irbr9os4uq1fl1vd12679du.apps.googleusercontent.com
✅ google_client_secret: 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

⏳ google_ga4_property_id:         (pending)
⏳ google_ga4_refresh_token:       (pending - auto-populated on first auth)
⏳ google_drive_service_account:   (pending)
⏳ google_drive_folder_id:         (pending)
⏳ google_ads_developer_token:     (pending)
⏳ google_ads_customer_id:         (pending)
⏳ google_ads_refresh_token:       (pending - auto-populated on first auth)
```

---

## References

### Google Cloud Resources
- [Google Cloud Console](https://console.cloud.google.com)
- [Google Drive API Documentation](https://developers.google.com/drive/api)
- [Google Analytics 4 API](https://developers.google.com/analytics/devguides/reporting/data/v1)
- [Google Ads API](https://developers.google.com/google-ads/api)

### Databricks Resources
- [Databricks Secrets API](https://docs.databricks.com/security/secrets/index.html)
- [Databricks SDK Documentation](https://docs.databricks.com/dev-tools/sdk-python.html)

### Project Resources
- [Connector Definitions](./connectors/)
- [Deployment Scripts](./scripts/deploy.py)
- [Deployment Template](./deployments/template/deployment.yaml)

---

**Last Updated:** July 17, 2026  
**Status:** Awaiting credential collection and connector deployment testing
