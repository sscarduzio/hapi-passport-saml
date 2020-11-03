# hapi-passport-saml
[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fsscarduzio%2Fhapi-passport-saml.svg?type=shield)](https://app.fossa.com/projects/git%2Bgithub.com%2Fsscarduzio%2Fhapi-passport-saml?ref=badge_shield)

A Hapi plugin that wraps passport-saml for SAML SSO (as SP)
with support for multiple strategies

## Current release
1.1.0

## Install

`npm install hapi-passport-saml`

## Configuration

Uses `samlidp.io` as IdP, read passport-saml for how to use options

```javascript
const idpCert = '...';
const decryptionCert = '...';
const samlOptions = {
  // passport saml settings
  saml: {
    callbackUrl: 'http://localhost/api/sso/v1/assert',
    logoutCallbackUrl: 'http://localhost/api/sso/v1/notifylogout',
    logoutUrl: 'https://my-idp.samlidp.io/saml2/idp/SingleLogoutService.php',
    host: 'localhost',
    protocol: 'http',
    entryPoint: 'https://my-idp.samlidp.io/saml2/idp/SSOService.php',
    // Service Provider Private Key
    decryptionPvk: fs.readFileSync(__dirname + '/private.key').toString(),
    // IdP Public Key
    cert: idpCert,
    issuer: 'my-saml'
  },
  // hapi-passport-saml settings
  config: {
    // Service Provider Public Key
    decryptionCert,
    // Plugin Routes
    routes: {
      // SAML Metadata
      metadata: {
        path: '/api/sso/v1/metadata.xml',
      },
      // SAML Assertion
      assert: {
        path: '/api/sso/v1/assert',
      },
    },
    assertHooks: {
      // Assertion Response Hook
      // Use this to add any specific props for your business
      // or appending to existing cookie
      onResponse: (profile) => {
        const username = profile['urn:oid:2.5.4.4'];
        return { ...profile, username };
      },
    }
  }
};

const serverPlugins = [{
  register: require('hapi-passport-saml'),
  options: samlOptions,
}];

// Internal cookie settings
const schemeOpts = {
  password: '14523695874159852035.0',
  isSecure: false,
  isHttpOnly: false,
  ttl: 3600,
}
server.register(serverPlugins, function (err) {
  server.auth.strategy('single-sign-on', 'saml', schemeOpts);
  server.register(controllers, {
    routes: {
      prefix: '/api'
    }
  }, function () {
    if (!module.parent) {
      server.start(function () {
        console.log('Server started at port ' + server.info.port);
      });
    }
  });

});
```

>Note: Internal cookie name is `hapi-passport-saml-cookie`, if you need to read the SAML credentials for integration with other strategies, use assertion hook.

## Multiple strategies

Use `hapi-passport-saml` as the last strategy. Tested with `try` and `required` modes.

* `required`: If successful, returns credentials, else HTTP 200 with JSON
* `try`: If successful, returns credentials, else empty credentials and isAuthenticated set to false

More info: [Integrating hapi cookie with hapi passport saml v1.1.0
](https://gist.github.com/molekilla/a7a899a3b3d7cbf2ae89998606102330)

## Demo application

[Demo](https://github.com/molekilla/hapi-passport-saml-test)

## References, Ideas and Based from
* [Saml2](https://github.com/Clever/saml2)
* [Passport-saml](https://github.com/bergie/passport-saml)

## License
MIT


[![FOSSA Status](https://app.fossa.com/api/projects/git%2Bgithub.com%2Fsscarduzio%2Fhapi-passport-saml.svg?type=large)](https://app.fossa.com/projects/git%2Bgithub.com%2Fsscarduzio%2Fhapi-passport-saml?ref=badge_large)