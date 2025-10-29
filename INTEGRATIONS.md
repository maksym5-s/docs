# Integrations Setup Guide

This guide explains how to set up and use the Nango-powered third-party integrations system for GoHighLevel, HubSpot, and other CRM platforms.

## Overview

The integration system allows users to connect their CRM accounts (GoHighLevel, HubSpot, Salesforce, Pipedrive) to sync data bidirectionally. The system uses Nango for OAuth flows and API management.

## Architecture

### Backend Components

1. **Database Model** (`Integration`): Stores integration connections and metadata
2. **API Endpoints** (`/api/v1/integrations`): Manages integration operations
3. **Controller** (`integrationController.ts`): Handles business logic
4. **Nango Service**: Manages OAuth flows and API calls

### Frontend Components

1. **Integrations Page**: Grid-based UI for managing integrations
2. **Integration Cards**: Display provider information and connection status
3. **OAuth Callback Handler**: Processes OAuth returns
4. **Nango Service**: Client-side integration management

## Setup Instructions

### 1. Nango Account Setup

1. Create a Nango account at [https://www.nango.dev](https://www.nango.dev)
2. Create a new project
3. Note your public and secret keys

### 2. Configure Integration Providers

In your Nango dashboard, add the following integrations:

#### GoHighLevel
- Integration ID: `gohighlevel`
- OAuth settings: Configure with GoHighLevel's OAuth endpoints
- Required scopes: `contacts.readonly`, `contacts.write`

#### HubSpot
- Integration ID: `hubspot`
- OAuth settings: Configure with HubSpot's OAuth endpoints
- Required scopes: `contacts`, `crm.objects.contacts.read`

#### Salesforce
- Integration ID: `salesforce`
- OAuth settings: Configure with Salesforce's OAuth endpoints
- Required scopes: `api`, `refresh_token`

#### Pipedrive
- Integration ID: `pipedrive`
- OAuth settings: Configure with Pipedrive's OAuth endpoints
- Required scopes: `base`

### 3. Environment Variables

#### Backend (.env)
```env
NANGO_SECRET_KEY=your_nango_secret_key
NANGO_HOST=https://api.nango.dev
FRONTEND_URL=http://localhost:3000
```

#### Frontend (.env)
```env
REACT_APP_NANGO_PUBLIC_KEY=your_nango_public_key
REACT_APP_NANGO_HOST=https://api.nango.dev
```

### 4. Database Setup

Run the Prisma migration to add the Integration model:

```bash
cd backend
npm run db:generate
npm run db:push
```

## API Endpoints

### GET /api/v1/integrations
Get all integrations for the authenticated company.

### GET /api/v1/integrations/providers
Get available integration providers with metadata.

### POST /api/v1/integrations/connect
Initiate OAuth flow for a provider.

**Request:**
```json
{
  "provider": "gohighlevel"
}
```

### POST /api/v1/integrations/callback
Handle OAuth callback (used internally).

### DELETE /api/v1/integrations/:id/disconnect
Disconnect an integration.

### GET /api/v1/integrations/:id/status
Get integration status and health.

### POST /api/v1/integrations/:id/sync
Manually trigger data synchronization.

## Frontend Usage

### Navigation

The integrations page is accessible from the dashboard sidebar with a "⚡ Integrations" menu item.

### User Flow

1. **Browse Available Integrations**: Users see a grid of available providers
2. **Connect Integration**: Click "Connect" to start OAuth flow
3. **OAuth Redirect**: User is redirected to provider's authorization page
4. **Callback Handling**: User returns to callback page for completion
5. **Manage Connections**: View connected integrations, sync, or disconnect

### Integration Cards

Each integration displays:
- Provider name and description
- Connection status (Connected, Disconnected, Error, Syncing)
- Last sync time
- Action buttons (Connect, Sync, Disconnect)
- Error messages if applicable

## Data Synchronization

### Supported Data Types

- **Contacts**: Bidirectional sync of contact information
- **Leads**: Import leads from CRM platforms
- **Activities**: Sync call logs and interactions
- **Custom Fields**: Map custom fields between systems

### Sync Triggers

1. **Manual Sync**: Users can trigger sync from the UI
2. **Scheduled Sync**: Background jobs sync data periodically
3. **Webhook Sync**: Real-time updates via provider webhooks

### Error Handling

- Connection errors are logged and displayed to users
- Failed syncs are retried with exponential backoff
- Users receive notifications for persistent errors

## Provider-Specific Implementation

### GoHighLevel

**Features:**
- Contact synchronization
- Lead import/export
- Campaign integration
- Call log sync

**API Endpoints Used:**
- `/contacts` - Contact management
- `/opportunities` - Lead tracking
- `/calls` - Call history

### HubSpot

**Features:**
- Contact and company sync
- Deal pipeline integration
- Email activity tracking
- Custom property mapping

**API Endpoints Used:**
- `/crm/v3/objects/contacts` - Contact management
- `/crm/v3/objects/deals` - Deal tracking
- `/crm/v3/objects/companies` - Company data

## Security Considerations

1. **OAuth Security**: All OAuth flows use state parameters and PKCE
2. **Token Storage**: Access tokens are securely stored in Nango's infrastructure
3. **API Rate Limits**: Built-in rate limiting and retry logic
4. **Audit Logging**: All integration activities are logged for security

## Monitoring and Analytics

### Integration Health

- Connection status monitoring
- Sync success/failure rates
- Error tracking and alerting

### Usage Analytics

- Popular integration providers
- Sync frequency and data volumes
- User engagement metrics

## Troubleshooting

### Common Issues

1. **OAuth Errors**: Check provider credentials and redirect URIs
2. **Sync Failures**: Verify API permissions and rate limits
3. **Connection Timeouts**: Review network connectivity and firewall settings

### Debug Mode

Enable debug logging by setting `NODE_ENV=development` in the backend environment.

### Support

For integration issues:
1. Check the integration status page
2. Review error logs in the dashboard
3. Contact support with integration ID and error details

## Development

### Adding New Providers

1. Configure provider in Nango dashboard
2. Add provider to `getAvailableProviders()` in controller
3. Implement provider-specific sync logic
4. Add provider logo and metadata
5. Test OAuth flow and data synchronization

### Testing

```bash
# Backend tests
cd backend
npm test integrations

# Frontend tests
cd frontend
npm test -- --testPathPattern=integrations
```

## Deployment

### Production Environment

1. Update environment variables with production Nango keys
2. Configure production OAuth redirect URIs
3. Set up monitoring and alerting
4. Schedule background sync jobs

### Scaling Considerations

- Database indexing for integration queries
- Queue system for async sync operations
- Load balancing for API endpoints
- Caching for frequently accessed integration data