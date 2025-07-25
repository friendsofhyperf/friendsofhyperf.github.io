# OAuth2 Server

A complete OAuth2 server implementation for the Hyperf framework based on [league/oauth2-server](https://oauth2.thephpleague.com/).

## Features

- Full OAuth2 server implementation supporting:
  - Client Credentials Grant
  - Password Grant
  - Refresh Token Grant
  - Authorization Code Grant (with PKCE support)
- Built-in client management commands
- Multiple storage backends (Eloquent ORM)
- Customizable token lifetimes
- Scope management
- Event-driven architecture

## Installation

### 1. Install via Composer

```bash
composer require friendsofhyperf/oauth2-server
```

### 2. Publish Configuration

```bash
php bin/hyperf.php vendor:publish friendsofhyperf/oauth2-server
```

### 3. Generate Encryption Keys

```bash
# Generate private/public key pair
php bin/hyperf.php oauth2:generate-keypair
```

This will generate:
- `storage/oauth2/private.key` - Private key for signing tokens
- `storage/oauth2/public.key` - Public key for verifying tokens

### 4. Run Database Migrations

```bash
php bin/hyperf.php migrate
```

## Configuration

Configure the OAuth2 server in `config/autoload/oauth2-server.php`:

```php
<?php

return [
    'authorization_server' => [
        'private_key' => env('OAUTH2_PRIVATE_KEY', 'storage/oauth2/private.key'),
        'private_key_passphrase' => env('OAUTH2_PRIVATE_KEY_PASSPHRASE'),
        'encryption_key' => env('OAUTH2_ENCRYPTION_KEY'),
        'encryption_key_type' => EncryptionKeyType::from(env('OAUTH2_ENCRYPTION_KEY_TYPE', 'plain')),
        'response_type' => BearerTokenResponse::class,
        'revoke_refresh_tokens' => true,
        'access_token_ttl' => new DateInterval('PT1H'),
        'auth_code_ttl' => new DateInterval('PT10M'),
        'refresh_token_ttl' => new DateInterval('P1M'),
        'enable_client_credentials_grant' => true,
        'enable_password_grant' => true,
        'enable_refresh_token_grant' => true,
        'enable_auth_code_grant' => true,
        'enable_implicit_grant' => false,
        'require_code_challenge_for_public_clients' => true,
        'persist_access_tokens' => true,
    ],
    'resource_server' => [
        'public_key' => env('OAUTH2_PUBLIC_KEY', 'storage/oauth2/public.key'),
        'jwt_leeway' => null,
    ],
    'scopes' => [
        'available' => ['read', 'write', 'admin'],
        'default' => ['read'],
    ],
];
```

## Environment Variables

Set these environment variables in your `.env` file:

```bash
# OAuth2 Keys
OAUTH2_PRIVATE_KEY=storage/oauth2/private.key
OAUTH2_PUBLIC_KEY=storage/oauth2/public.key
OAUTH2_PRIVATE_KEY_PASSPHRASE=
OAUTH2_ENCRYPTION_KEY=your-encryption-key-here

# Optional
OAUTH2_ENCRYPTION_KEY_TYPE=plain
```

## Available Commands

| Command | Description |
|---------|-------------|
| `oauth2:clear-expired-tokens` | Clear expired access/refresh tokens |
| `oauth2:create-client` | Create new OAuth2 client |
| `oauth2:delete-client` | Delete OAuth2 client |
| `oauth2:generate-keypair` | Generate private/public key pair |
| `oauth2:list-clients` | List all OAuth2 clients |
| `oauth2:update-client` | Update OAuth2 client |

### Create Client

Create client for authorization code grant:

```bash
php bin/hyperf.php oauth2:create-client \
    --name="My Web App" \
    --redirect-uri="https://myapp.com/callback" \
    --grant-type="authorization_code" \
    --grant-type="refresh_token"
```

Create client for password grant:

```bash
php bin/hyperf.php oauth2:create-client \
    --name="My Mobile App" \
    --grant-type="password" \
    --grant-type="refresh_token"
```

Create client for client credentials grant:

```bash
php bin/hyperf.php oauth2:create-client \
    --name="My API Service" \
    --grant-type="client_credentials"
```

## API Endpoints

### Authorization Endpoint

`GET /oauth/authorize`

For authorization code flow. Parameters:
- `response_type`: Must be `code`
- `client_id`: Client ID
- `redirect_uri`: Must match registered callback URI
- `scope`: Space-separated scope list
- `state`: CSRF protection token
- `code_challenge`: PKCE code challenge
- `code_challenge_method`: PKCE method (usually `S256`)

### Token Endpoint

`POST /oauth/token`

For exchanging authorization codes for access tokens or using other grant types.

### Protected Resources

Protect routes with `ResourceServerMiddleware`:

```php
use FriendsOfHyperf\Oauth2\Server\Middleware\ResourceServerMiddleware;

Router::addGroup('/api', function () {
    Router::get('user', [UserController::class, 'index']);
    Router::post('posts', [PostController::class, 'store']);
})->add(ResourceServerMiddleware::class);
```

## Grant Types

### 1. Client Credentials Grant

For server-to-server authentication:

```bash
curl -X POST http://your-server/oauth/token \
    -H "Content-Type: application/json" \
    -d '{
        "grant_type": "client_credentials",
        "client_id": "your-client-id",
        "client_secret": "your-client-secret",
        "scope": "read write"
    }'
```

### 2. Password Grant

For trusted applications (mobile apps, SPAs):

```bash
curl -X POST http://your-server/oauth/token \
    -H "Content-Type: application/json" \
    -d '{
        "grant_type": "password",
        "client_id": "your-client-id",
        "client_secret": "your-client-secret",
        "username": "user@example.com",
        "password": "password",
        "scope": "read write"
    }'
```

### 3. Authorization Code Grant

For web applications requiring user interaction:

**Step 1: Redirect user to authorization endpoint**

```
https://your-server/oauth/authorize?response_type=code&client_id=your-client-id&redirect_uri=https://myapp.com/callback&scope=read&state=random-state&code_challenge=challenge&code_challenge_method=S256
```

**Step 2: Exchange authorization code for token**

```bash
curl -X POST http://your-server/oauth/token \
    -H "Content-Type: application/json" \
    -d '{
        "grant_type": "authorization_code",
        "client_id": "your-client-id",
        "client_secret": "your-client-secret",
        "redirect_uri": "https://myapp.com/callback",
        "code_verifier": "verifier",
        "code": "authorization-code-from-redirect"
    }'
```

### 4. Refresh Token Grant

Obtain new access token:

```bash
curl -X POST http://your-server/oauth/token \
    -H "Content-Type: application/json" \
    -d '{
        "grant_type": "refresh_token",
        "client_id": "your-client-id",
        "client_secret": "your-client-secret",
        "refresh_token": "your-refresh-token",
        "scope": "read write"
    }'
```

## Making Authenticated Requests

Include access token in Authorization header:

```bash
curl -X GET http://your-server/api/user \
    -H "Authorization: Bearer your-access-token"
```

## Event System

The component dispatches these events you can listen for:

- `AuthorizationRequestResolveEvent`: When authorization request needs user approval
- `UserResolveEvent`: When resolving user for password grant
- `ScopeResolveEvent`: When resolving scopes
- `TokenRequestResolveEvent`: When processing token requests

### Example Event Listener

```php
<?php

namespace App\Listener;

use FriendsOfHyperf\Oauth2\Server\Event\UserResolveEvent;
use Hyperf\Event\Annotation\Listener;

#[Listener]
class UserResolveListener
{
    public function listen(): array
    {
        return [
            UserResolveEvent::class,
        ];
    }

    public function process(object $event): void
    {
        // Validate user credentials and return user ID
        if ($event->getUsername() === 'admin' && $event->getPassword() === 'secret') {
            $event->setUserId('1');
        }
    }
}
```

## Database Tables

The package creates these tables:

- `oauth_clients`: OAuth2 clients
- `oauth_access_tokens`: Access tokens
- `oauth_refresh_tokens`: Refresh tokens
- `oauth_auth_codes`: Authorization codes
- `oauth_personal_access_clients`: Personal access clients

## Customization

### Custom User Provider

Implement your own user resolution logic by listening to `UserResolveEvent`.

### Custom Scope Management

Listen to `ScopeResolveEvent` to implement custom scope logic.

### Custom Token Storage

Extend repository classes to implement custom storage backends.

## Security Best Practices

1. Always use HTTPS in production
2. Store private keys securely with proper file permissions
3. Use strong encryption keys
4. Implement proper CSRF protection for authorization flow
5. Strictly validate callback URIs
6. Use short-lived access tokens with refresh tokens
7. Implement rate limiting on token endpoint
8. Log and monitor token usage

## Testing

During development, you can test OAuth2 flows using built-in commands:

```bash
# Create test client
php bin/hyperf.php oauth2:create-client \
    --name="Test Client" \
    --redirect-uri="http://localhost:3000/callback" \
    --grant-type="authorization_code" \
    --grant-type="password" \
    --grant-type="refresh_token"

# List all clients
php bin/hyperf.php oauth2:list-clients

# Clear expired tokens
php bin/hyperf.php oauth2:clear-expired-tokens
```

## Error Handling

Common error responses:

- `invalid_client`: Client authentication failed
- `invalid_grant`: Invalid grant
- `invalid_request`: Missing required parameters
- `invalid_scope`: Requested scope is invalid
- `unsupported_grant_type`: Unsupported grant type
- `server_error`: Internal server error

## License

MIT