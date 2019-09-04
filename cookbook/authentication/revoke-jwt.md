# Revoking JWTs

The ability to revoke access tokens can be added by customizing [the authentication service](../../api/authentication/service.md).

## Authentication service

The following example stores all revoked tokens in a plain object. In most applications you would use something like e.g. Redis with node_redis to store them more permanently.

```js
const { AuthenticationService } = require('@feathersjs/authentication');
const { NotAuthenticated } = require('@feathersjs/errors');

const revokedTokens = {};

class RevokableAuthService extends AuthenticationService {
  async revokeAccessToken (accessToken) {
    // First make sure the access token is valid
    const verified = await this.verifyAccessToken(accessToken);

    revokedTokens[accessToken] = true;

    return verified;
  }

  async verifyAccessToken(accessToken) {
    // First check if the token has been revoked
    if (revokedTokens[accessToken]) {
      throw new NotAuthenticated('Token revoked');
    }

    return super.verifyAccessToken(accessToken);
  }

  async remove (id, params) {
    const authResult = await super.remove(id, params);
    const { accessToken } = authResult;

    if (accessToken) {
      // If there is an access token, revoke it
      await this.revokeAccessToken(accessToken);
    }

    return authResult;
  }
}

app.use('/authentication', new MyAuthService(app));
```
