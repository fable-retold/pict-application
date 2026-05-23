# Pict-Application

> Application base class for building pict view-based applications

Pict-Application provides a structured foundation for building web, console, and other applications where the UI is primarily represented as text strings. It extends the Pict ecosystem by providing application lifecycle management, view coordination, and data marshaling capabilities.

## Features

- **Lifecycle Management** - Complete initialization, solving, rendering, and marshaling lifecycles
- **View Coordination** - Manages multiple views with automatic rendering and solving
- **Data Marshaling** - Two-way data binding between views and application state
- **Provider Integration** - Coordinates data providers for loading and saving
- **Login Support** - Built-in authentication lifecycle hooks
- **Async Support** - Both synchronous and asynchronous lifecycle methods

## Quick Start

```javascript
const libPict = require('pict');
const libPictApplication = require('pict-application');

// Create a Pict instance
const _Pict = new libPict();

// Add an application
const myApp = _Pict.addApplication('MyApp', {
    Name: 'My Application',
    AutoRenderMainViewportViewAfterInitialize: true
}, libPictApplication);

// Initialize the application
myApp.initialize();
```

## Installation

```bash
npm install pict-application
```

## Configuration

### Default Settings

```javascript
const defaultPictSettings = {
    Name: 'DefaultPictApplication',

    // Main viewport configuration
    MainViewportViewIdentifier: 'Default-View',
    MainViewportRenderableHash: false,
    MainViewportDestinationAddress: false,
    MainViewportDefaultDataAddress: false,

    // Auto behaviors
    AutoSolveAfterInitialize: true,
    AutoRenderMainViewportViewAfterInitialize: true,
    AutoRenderViewsAfterInitialize: false,
    AutoLoginAfterInitialize: false,
    AutoLoadDataAfterLogin: false,

    // Views and Manifests
    ConfigurationOnlyViews: [],
    Manifests: {},

    // Template addressing
    IdentifierAddressPrefix: 'PICT-'
};
```

### Configuration Options

| Option | Type | Default | Description |
|--------|------|---------|-------------|
| `Name` | string | 'DefaultPictApplication' | Application name for logging |
| `MainViewportViewIdentifier` | string | 'Default-View' | Default view to render |
| `MainViewportRenderableHash` | string/boolean | false | Default renderable hash |
| `MainViewportDestinationAddress` | string/boolean | false | DOM selector for main content |
| `MainViewportDefaultDataAddress` | string/boolean | false | Data address for templates |
| `AutoSolveAfterInitialize` | boolean | true | Run solve after init |
| `AutoRenderMainViewportViewAfterInitialize` | boolean | true | Render main view after init |
| `AutoRenderViewsAfterInitialize` | boolean | false | Render all auto-render views |
| `AutoLoginAfterInitialize` | boolean | false | Trigger login flow on init |
| `AutoLoadDataAfterLogin` | boolean | false | Load data after login |
| `ConfigurationOnlyViews` | array | [] | Views defined by config only |
| `Manifests` | object | {} | Data transformation manifests |

### Carried Over Configuration

Configuration can be passed through the Pict settings:

```javascript
const _Pict = new libPict({
    PictApplicationConfiguration: {
        CustomValue: 'MyValue',
        AutoRenderMainViewportViewAfterInitialize: false
    }
});
```


## Application Lifecycle

Pict Application follows a structured lifecycle with hooks at each stage:

### Initialization Phase

```
onBeforeInitialize() -> onInitialize() -> [Provider Init] -> [View Init] -> onAfterInitialize()
    ↓
[AutoSolve] -> [AutoRender] -> onCompletionOfInitialize()
```

### Solve Phase

The solve phase processes data and prepares views for rendering:

```
onPreSolve() -> onBeforeSolve() -> [Provider Solve] -> [View Solve] -> onSolve() -> onAfterSolve()
```

### Render Phase

```
onBeforeRender() -> onRender() -> [View Render] -> onAfterRender()
```

### Login Phase

```
onBeforeLoginAsync() -> onLoginAsync() -> onAfterLoginAsync() -> [AutoLoadData]
```

### Data Loading Phase

```
onBeforeLoadDataAsync() -> [Provider Load] -> onLoadDataAsync() -> onAfterLoadDataAsync()
```

### Data Saving Phase

```
onBeforeSaveDataAsync() -> [Provider Save] -> onSaveDataAsync() -> onAfterSaveDataAsync()
```

### Marshal Phases

Data marshaling between views and AppData:

```
// From Views -> AppData
onBeforeMarshalFromViews() -> [View Marshal] -> onMarshalFromViews() -> onAfterMarshalFromViews()

// From AppData -> Views
onBeforeMarshalToViews() -> [View Marshal] -> onMarshalToViews() -> onAfterMarshalToViews()
```


## Creating an Application

### Basic Application

```javascript
const libPictApplication = require('pict-application');

const defaultConfig = {
    Name: 'Simple Test Application'
};

class SimpleApplication extends libPictApplication {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }
}

module.exports = SimpleApplication;
module.exports.default_configuration = defaultConfig;
```

### Application with Custom Initialization

```javascript
class CustomApplication extends libPictApplication {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }

    onInitialize() {
        super.onInitialize();
        // Custom initialization logic
        this.pict.AppData.CustomState = { initialized: true };
        return true;
    }

    onAfterInitialize() {
        super.onAfterInitialize();
        this.log.info('Application fully initialized');
        return true;
    }
}
```

### Application with Login

```javascript
class AuthenticatedApplication extends libPictApplication {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
        this._isAuthenticated = false;
    }

    isLoggedIn() {
        return this._isAuthenticated;
    }

    onLoginAsync(fCallback) {
        // Perform authentication logic
        this.log.info('Checking authentication...');

        // Example: Check for stored token
        if (this.pict.AppData.AuthToken) {
            this._isAuthenticated = true;
        }

        return fCallback();
    }
}
```


## Working with Views

### Adding Views

```javascript
const libPictView = require('pict-view');

// Add a view to the application
_Pict.addView('MyView', {
    ViewIdentifier: 'MyView',
    DefaultRenderable: 'Main-Content',
    DefaultDestinationAddress: '#app-container',
    AutoInitialize: true,
    AutoRender: true
}, libPictView);
```

### View Configuration

```javascript
const viewConfig = {
    ViewIdentifier: 'UserProfile',
    DefaultRenderable: 'Profile-Main',
    DefaultDestinationAddress: '#user-profile',
    DefaultTemplateRecordAddress: 'AppData.CurrentUser',

    AutoInitialize: true,
    AutoInitializeOrdinal: 0,
    AutoRender: true,
    AutoRenderOrdinal: 0,
    AutoSolveWithApp: true,
    AutoSolveOrdinal: 0,

    Templates: [
        {
            Hash: 'Profile-Template',
            Template: /*html*/`
                <div class="user-profile">
                    <h1>{~Data:Record.name~}</h1>
                    <p>{~Data:Record.bio~}</p>
                </div>
            `
        }
    ],

    Renderables: [
        {
            RenderableHash: 'Profile-Main',
            TemplateHash: 'Profile-Template',
            DestinationAddress: '#user-profile',
            RenderMethod: 'replace'
        }
    ]
};
```

### Custom View with Solver

```javascript
const libPictView = require('pict-view');

class CustomView extends libPictView {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }

    onBeforeSolve() {
        super.onBeforeSolve();
        // Initialize state if needed
        if (!('SolveCount' in this.pict.AppData)) {
            this.pict.AppData.SolveCount = 0;
        }
    }

    onSolve() {
        super.onSolve();
        this.pict.AppData.SolveCount++;
    }

    onAfterSolve() {
        this.pict.log.info(`SolveCount: ${this.pict.AppData.SolveCount}`);
    }
}

module.exports = CustomView;
```


## Renderables

Renderables define how templates are rendered to the DOM.

### Renderable Structure

```javascript
{
    RenderableHash: 'Content-List',       // Unique identifier
    TemplateHash: 'Content-Template',     // Template to render
    TemplateRecordAddress: 'AppData.Items', // Data source
    DestinationAddress: '#container',     // DOM target
    RenderMethod: 'replace'               // How to insert content
}
```

### Render Methods

| Method | Description |
|--------|-------------|
| `replace` | Replace content at destination outright |
| `append` | Append content after existing content |
| `prepend` | Prepend content before existing content |
| `append_once` | Append only once (checks for existing ID) |
| `virtual-assignment` | Deferred assignment for complex transactions |

### Adding Renderables at Runtime

```javascript
// Method signature
myView.addRenderable(
    pRenderableHash,
    pTemplateHash,
    pDefaultTemplateDataAddress,
    pDefaultDestinationAddress,
    pRenderMethod
);

// Example
myView.addRenderable(
    'Dynamic-Content',
    'Dynamic-Template',
    'AppData.DynamicData',
    '#dynamic-container',
    'append'
);

// Or pass a renderable object
myView.addRenderable({
    RenderableHash: 'Dynamic-Content',
    TemplateHash: 'Dynamic-Template',
    TemplateRecordAddress: 'AppData.DynamicData',
    DestinationAddress: '#dynamic-container',
    RenderMethod: 'append'
});
```


## Providers

Providers are services that can participate in the application lifecycle.

### Creating a Provider

```javascript
const libPictProvider = require('pict-provider');

class DataProvider extends libPictProvider {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }

    onBeforeInitializeAsync(fCallback) {
        super.onBeforeInitialize();
        // Load configuration, setup connections, etc.
        return fCallback();
    }

    onLoadDataAsync(fCallback) {
        // Fetch data from API
        fetch('/api/data')
            .then(response => response.json())
            .then(data => {
                this.pict.AppData.ExternalData = data;
                fCallback();
            })
            .catch(fCallback);
    }

    onSaveDataAsync(fCallback) {
        // Save data to API
        fetch('/api/data', {
            method: 'POST',
            body: JSON.stringify(this.pict.AppData.ExternalData)
        })
        .then(() => fCallback())
        .catch(fCallback);
    }
}

module.exports = DataProvider;
```

### Provider Configuration

```javascript
_Pict.addProvider('DataProvider', {
    AutoInitialize: true,
    AutoInitializeOrdinal: 0,
    AutoSolveWithApp: true,
    AutoSolveOrdinal: 0,
    AutoLoadDataWithApp: true,
    AutoLoadDataOrdinal: 0,
    AutoSaveDataWithApp: true,
    AutoSaveDataOrdinal: 0
}, DataProvider);
```


## API Reference

### Properties

| Property | Type | Description |
|----------|------|-------------|
| `pict` | object | Reference to the Pict instance |
| `fable` | object | Reference to the Fable instance |
| `AppData` | object | Application data store |
| `Bundle` | object | Pict bundle reference |
| `UUID` | string | Unique identifier |
| `Hash` | string | Service hash |
| `options` | object | Configuration options |
| `log` | object | Logger instance |
| `initializeTimestamp` | number | When initialized |
| `lastSolvedTimestamp` | number | Last solve time |
| `lastLoginTimestamp` | number | Last login time |
| `lastMarshalFromViewsTimestamp` | number | Last marshal from views |
| `lastMarshalToViewsTimestamp` | number | Last marshal to views |
| `lastAutoRenderTimestamp` | number | Last auto render |
| `lastLoadDataTimestamp` | number | Last data load |

### Synchronous Methods

| Method | Returns | Description |
|--------|---------|-------------|
| `initialize()` | boolean | Initialize the application |
| `solve()` | boolean | Run solve phase |
| `render(viewId?, renderableHash?, destAddress?, dataAddress?)` | boolean | Render a view |
| `renderMainViewport()` | boolean | Render main viewport |
| `renderAutoViews()` | void | Render all auto-render views |
| `marshalFromViews()` | boolean | Marshal data from all views |
| `marshalToViews()` | boolean | Marshal data to all views |
| `isLoggedIn()` | boolean | Check login state |

### Asynchronous Methods

| Method | Description |
|--------|-------------|
| `initializeAsync(callback)` | Async initialization |
| `solveAsync(callback)` | Async solve phase |
| `renderAsync(viewId?, renderableHash?, destAddress?, dataAddress?, callback)` | Async render |
| `renderMainViewportAsync(callback)` | Async render main viewport |
| `renderAutoViewsAsync(callback)` | Async render auto views |
| `loginAsync(callback)` | Async login flow |
| `loadDataAsync(callback)` | Async data loading |
| `saveDataAsync(callback)` | Async data saving |
| `marshalFromViewsAsync(callback)` | Async marshal from views |
| `marshalToViewsAsync(callback)` | Async marshal to views |

### Lifecycle Hooks

All lifecycle hooks have both sync and async variants:

**Initialization:**
- `onBeforeInitialize()` / `onBeforeInitializeAsync(callback)`
- `onInitialize()` / `onInitializeAsync(callback)`
- `onAfterInitialize()` / `onAfterInitializeAsync(callback)`
- `onCompletionOfInitialize()` / `onCompletionOfInitializeAsync(callback)`

**Solve:**
- `onPreSolve()` / `onPreSolveAsync(callback)`
- `onBeforeSolve()` / `onBeforeSolveAsync(callback)`
- `onSolve()` / `onSolveAsync(callback)`
- `onAfterSolve()` / `onAfterSolveAsync(callback)`

**Render:**
- `onBeforeRender()` / `onBeforeRenderAsync(callback)`
- `onRender()` / `onRenderAsync(callback)`
- `onAfterRender()` / `onAfterRenderAsync(callback)`

**Login:**
- `onBeforeLoginAsync(callback)`
- `onLoginAsync(callback)`
- `onAfterLoginAsync(callback)`

**Data Loading:**
- `onBeforeLoadDataAsync(callback)`
- `onLoadDataAsync(callback)`
- `onAfterLoadDataAsync(callback)`

**Data Saving:**
- `onBeforeSaveDataAsync(callback)`
- `onSaveDataAsync(callback)`
- `onAfterSaveDataAsync(callback)`

**Marshal:**
- `onBeforeMarshalFromViews()` / `onBeforeMarshalFromViewsAsync(callback)`
- `onMarshalFromViews()` / `onMarshalFromViewsAsync(callback)`
- `onAfterMarshalFromViews()` / `onAfterMarshalFromViewsAsync(callback)`
- `onBeforeMarshalToViews()` / `onBeforeMarshalToViewsAsync(callback)`
- `onMarshalToViews()` / `onMarshalToViewsAsync(callback)`
- `onAfterMarshalToViews()` / `onAfterMarshalToViewsAsync(callback)`


## Template Syntax

Pict uses a template syntax with special markers:

| Syntax | Description | Example |
|--------|-------------|---------|
| `{~Data:path~}` | Output data value | `{~Data:Record.name~}` |
| `{~TS:hash:address~}` | Template set (loop) | `{~TS:Row-Template:AppData.Items~}` |
| `{~Digits:path~}` | Format as digits | `{~Digits:Record.count~}` |


## Examples

### Example Applications

The repository includes several example applications:

1. **Simple Example** - Basic application setup
2. **Bookstore Example** - Views with external data
3. **Historic Events Example** - Complex data marshaling
4. **Postcard Example** - Navigation and multiple views

### Running Examples

```bash
# Install dependencies
npm install

# Navigate to examples folder
cd example_applications

# Build examples
./Build-Examples.sh

# Serve examples
node ServeExamples.js
```


## Testing

```bash
# Run all tests
npm test

# Run specific test
npm run tests -- "Object Initialization"

# Run with coverage
npm run coverage
```


## Building

```bash
# Build distribution files
npm run build

# Generate TypeScript declarations
npm run types

# Lint source code
npm run lint
```


## Related Packages

- [pict](https://github.com/stevenvelozo/pict) - Core Pict framework
- [pict-view](https://github.com/stevenvelozo/pict-view) - View base class
- [pict-provider](https://github.com/stevenvelozo/pict-provider) - Provider base class
- [fable](https://github.com/stevenvelozo/fable) - Service provider framework
- [quackage](https://github.com/stevenvelozo/quackage) - Build tooling


## License

MIT License - see [LICENSE](https://github.com/stevenvelozo/pict-application/blob/main/LICENSE) for details.
