---
title: Authorization
description: This tutorial demonstrates how to add authorization and access control to an Angular 2+ app with Auth0
budicon: 546
---

<%= include('../../../_includes/_package', {
  org: 'auth0-samples',
  repo: 'auth0-angular-samples',
  path: '04-Authorization',
  requirements: [
    'Angular 2+'
  ]
}) %>

<%= include('../_includes/_authz_preamble') %>

<%= include('../_includes/_authz_determining_scopes') %>

## Handle Scopes in the `AuthService`

Adjust your `AuthService` to use a local member with any `scope`s you would like to request when users log in. Use this member in the `auth0.WebAuth` instance.

```ts
// src/app/auth/auth.service.ts

requestedScopes: string = 'openid profile read:messages write:messages';

auth0 = new auth0.WebAuth({
  // ...
  scope: this.requestedScopes
});
``` 

In the `setSession` method, save the `scope`s granted for the user into local storage. The first place to check for these granted `scope` values is the `scope` key from the `authResult`. If something exists there it's because the `scope`s which were granted for the user differ from those that were requested. If there is nothing on `authResult.scope`, it means that the granted `scope`s match those that were requested, so the requested values can be used directly. If there are no values for either of these, you can fall back to an empty string.

```ts
// src/app/auth/auth.service.ts

private setSession(authResult): void {

  const scopes = authResult.scope || this.requestedScopes || '';

  // ...
  localStorage.setItem('scopes', JSON.stringify(scopes));
}
```

Add a method called `userHasScopes` which will check for a particular `scope` in local storage. This method should take an array of strings and check whether the array of `scope`s saved in local storage contains those values. This method can be used to conditionally hide and show various UI elements and to limit route access.

```ts
// src/app/auth/auth.service.ts

public userHasScopes(scopes: Array<string>): boolean {
  const grantedScopes = JSON.parse(localStorage.getItem('scopes')).split(' ');
  return scopes.every(scope => grantedScopes.includes(scope));
}
```

## Conditionally Dislay UI Elements

The `userHasScopes` method can now be used alongside `isAuthenticated` to conditionally show and hide certain UI elements based on those two conditions.

```html
<!-- src/app/app.component.html -->

<button
  class="btn btn-primary btn-margin"
  *ngIf="auth.isAuthenticated() && auth.userHasScopes(['write:messages'])"
  routerLink="/admin">
    Admin Area
</button>
```

## Protect Client-Side Routes

To completely prevent access to client-side routes based on a particular `scope`, use an `AuthGuard`.

```ts
// src/app/auth/auth-guard.service.ts

import { Injectable } from '@angular/core';
import { Router, CanActivate } from '@angular/router';
import { AuthService } from './auth.service';

@Injectable()
export class AuthGuardService implements CanActivate {

  constructor(public auth: AuthService, public router: Router) {}

  canActivate(): boolean {
    if (this.auth.isAuthenticated()) {
      if (this.auth.userHasScopes(['write:messages'])) {
        return true;
      } else {
        this.router.navigate(['']);
        return false;
      }
    }
  }

}
```

The guard implements the `CanActivate` interface which requires a method called `canActivate` on the service. This method returns `true` if the user is authenticated and has a `scope` of `write:messages`, and `false` if not. It also navigates the user to the home route if they don't have the appropriate `scope` to access the route.

Use the `AuthGuard` in the routing definition.

```ts
// src/app/app.routes.ts

import { AuthGuardService as AuthGuard } from './auth/auth-guard.service';

export const ROUTES: Routes = [
  // ...
  { path: 'admin', component: AdminComponent, canActivate: [AuthGuard] }
];
```

The user will now be redirected to the main route unless they have a `scope` of `write:messages`.

<%= include('../_includes/_authz_conditionally_assign_scopes') %>

<%= include('../_includes/_authz_client_routes_disclaimer') %>