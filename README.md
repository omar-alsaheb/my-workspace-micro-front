# MyWorkspace

This project was generated using [Angular CLI](https://github.com/angular/angular-cli) version 19.1.4.

## Development Server

To start a local development server, run:

```bash
ng serve
```

Once the server is running, open your browser and navigate to `http://localhost:4200/`. The application will automatically reload whenever you modify any source files.

## Code Scaffolding

Angular CLI includes powerful code scaffolding tools. To generate a new component, run:

```bash
ng generate component component-name
```

For a complete list of available schematics (such as `components`, `directives`, or `pipes`), run:

```bash
ng generate --help
```

## Building the Project

To build the project, run:

```bash
ng build
```

This will compile the project and store the build artifacts in the `dist/` directory. The production build optimizes your application for performance and speed.

## Running Unit Tests

To execute unit tests with the [Karma](https://karma-runner.github.io) test runner, use:

```bash
ng test
```

## Running End-to-End Tests

For end-to-end (e2e) testing, run:

```bash
ng e2e
```

Note: Angular CLI does not include an end-to-end testing framework by default. You can choose and configure one that fits your needs.

## Micro-Frontend Setup with Module Federation

This project is structured as a Micro-Frontend (MFE) using [Module Federation](https://webpack.js.org/concepts/module-federation/) for Angular applications.

### Creating a Workspace

```bash
ng new my-workspace --create-application=false
```

### Generating a Remote Application

```bash
ng generate application remote --prefix app-remote --no-standalone --style=scss
```

Navigate to the remote application:

```bash
cd remote
```

### Adding Module Federation to Remote

```bash
ng add @angular-architects/module-federation --project remote --port 4201 --type remote
```

### Creating a Remote Module

```bash
ng g module remoteMain
```

Navigate to the newly created module:

```bash
cd remote-main
```

Generate a component inside the remote module:

```bash
ng g c home
```

### Configuring Webpack for Remote

Modify `webpack.config.js`:

```javascript
const { shareAll, withModuleFederationPlugin } = require('@angular-architects/module-federation/webpack');

module.exports = withModuleFederationPlugin({
  name: 'remote',
  exposes: {
    './RemoteMainModule': './projects/remote/src/app/remote-main/remote-main.module.ts',
  },
  shared: {
    ...shareAll({ singleton: true, strictVersion: true, requiredVersion: 'auto' }),
  },
});
```

### Configuring Routing for Remote

Modify `remote-main/app-routing.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';

const routes: Routes = [
  {
    path: '',
    loadChildren: () => import('./remote-main/remote-main.module').then(m => m.RemoteMainModule)
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

### Running the Remote Application

```bash
npm run start
```

---

## Setting Up the Shell Application

### Generating the Shell Application

```bash
ng generate application shell --prefix app-shell --no-standalone --style=scss
```

### Adding Module Federation to Shell

```bash
ng add @angular-architects/module-federation --project shell --port 4200 --type dynamic-host
```

### Configuring Module Federation Manifest

Create `assets/mf.manifest.json`:

```json
{
  "remote": "http://localhost:4201/remoteEntry.js"
}
```

### Creating a Home Component in Shell

```bash
ng g c home-shell
```

### Configuring Routing for Shell

Modify `app-routing.module.ts`:

```typescript
import { NgModule } from '@angular/core';
import { RouterModule, Routes } from '@angular/router';
import { HomeShellComponent } from './home-shell/home-shell.component';
import { loadRemoteModule } from '@angular-architects/module-federation';

const routes: Routes = [
  {
    path: '',
    component: HomeShellComponent,
    pathMatch: 'full',
  },
  {
    path: 'remote',
    loadChildren: () =>
      loadRemoteModule({
        type: 'module',
        remoteEntry: 'http://localhost:4201/remoteEntry.js',
        exposedModule: './RemoteMainModule',
      }).then(m => m.RemoteMainModule),
  }
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule]
})
export class AppRoutingModule {}
```

### Updating the Shell Application's Layout

Modify `app.component.html`:

```html
<ul>
  <li><a routerLinkActive="active" routerLink="/">Home</a></li>
  <li><a routerLink="/remote">Remote Application</a></li>
</ul>

<router-outlet></router-outlet>
```

### Running the Applications

Start the shell application:

```bash
ng serve shell
```

Start the remote application:

```bash
ng serve remote
```

## Additional Resources

For more information on using the Angular CLI, visit the [Angular CLI Overview and Command Reference](https://angular.dev/tools/cli).

