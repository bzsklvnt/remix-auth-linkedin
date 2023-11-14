# LinkedinStrategy

The Linkedin strategy is used to authenticate users against a Linkedin account. It extends the [OAuth2Strategy](https://github.com/sergiodxa/remix-auth-oauth2).

## Attribution and Acknowledgments
This project is a fork of [remix-auth-linkedin](https://github.com/Lzok/remix-auth-linkedin) by [juhanakristian](https://github.com/Lzok). We are immensely grateful to the original authors for their hard work and dedication in creating a foundation that has inspired our version. Our modifications and extensions are built upon the robust groundwork they established.

For details about the original project and its features, please visit the Original Repository.

We aim to respect the original vision while also adding our unique contributions, which are documented in our changelog and release notes. Our project follows the licensing terms set forth in the original, adhering to the principles of open-source collaboration and mutual respect for the creators' efforts.

We encourage users and contributors to also explore the original repository to understand the origins of this project and appreciate the collective effort involved in its ongoing development.

In this version we aimed to make changes, so that this project is compatible with @remix-run/server-runtime v2 >.

## Supported runtimes

| Runtime    | Has Support |
| ---------- | ----------- |
| Node.js    | ✅          |
| Cloudflare | ✅          |


## Usage

### Create an OAuth application

First you need to create a new application in the [Linkedin's developers page](https://developer.linkedin.com/). Then I encourage you to read [this documentation page](https://learn.microsoft.com/en-us/linkedin/consumer/integrations/self-serve/sign-in-with-linkedin-v2), it explains how to configure your app and gives you useful information on the auth flow.
The app is mandatory in order to obtain a `clientID` and `client secret` to use with the Linkedin's API.

### Create the strategy instance

```ts
// linkedin.server.ts
import { createCookieSessionStorage } from 'remix';
import { Authenticator } from 'remix-auth';
import { LinkedinStrategy } from "remix-auth-linkedin";

// Personalize this options for your usage.
const cookieOptions = {
	path: '/',
	httpOnly: true,
	sameSite: 'lax' as const,
	maxAge: 24 * 60 * 60 * 1000 * 30,
	secrets: ['THISSHOULDBESECRET_AND_NOT_SHARED'],
	secure: process.env.NODE_ENV !== 'development',
};

const sessionStorage = createCookieSessionStorage({
	cookie: cookieOptions,
});

export const authenticator = new Authenticator<string>(sessionStorage, {
	throwOnError: true,
});

const linkedinStrategy = new LinkedinStrategy(
   {
      clientID: "YOUR_CLIENT_ID",
      clientSecret: "YOUR_CLIENT_SECRET",
      // LinkedIn is expecting a full URL here, not a relative path
      // see: https://learn.microsoft.com/en-us/linkedin/shared/authentication/authorization-code-flow?tabs=HTTPS1#step-1-configure-your-application
      callbackURL: "https://example.com/auth/linkedin/callback";
   },
   async ({accessToken, refreshToken, extraParams, profile, context}) => {
      /*
         profile:
         type LinkedinProfile = {
            id: string;
            displayName: string;
            name: {
               givenName: string;
               familyName: string;
            };
            emails: Array<{ value: string }>;
            photos: Array<{ value: string }>;
            _json: LiteProfileData & EmailData;
         } & OAuth2Profile;
      */

      // Get the user data from your DB or API using the tokens and profile
      return User.findOrCreate({ email: profile.emails[0].value });
   }
);

authenticator.use(linkedinStrategy, 'linkedin');
```

### Setup your routes

```tsx
// app/routes/login.tsx
export default function Login() {
   return (
      <Form action="/auth/linkedin" method="post">
         <button>Login with Linkedin</button>
      </Form>
   )
}
```

```tsx
// app/routes/auth/linkedin.tsx
import { ActionFunction, LoaderFunction } from 'remix'
import { authenticator } from '~/linkedin.server'

export let loader: LoaderFunction = () => redirect('/login')
export let action: ActionFunction = ({ request }) => {
   return authenticator.authenticate('linkedin', request)
}
```

```tsx
// app/routes/auth/linkedin/callback.tsx
import { ActionFunction, LoaderFunction } from 'remix'
import { authenticator } from '~/linkedin.server'

export let loader: LoaderFunction = ({ request }) => {
   return authenticator.authenticate('linkedin', request, {
      successRedirect: '/dashboard',
      failureRedirect: '/login',
   })
}
```
