# Getting Started with Pict Application

This guide will help you create your first Pict Application from scratch.

---

## Prerequisites

Before starting, ensure you have:

- Node.js 14+ installed
- npm or yarn package manager
- Basic understanding of JavaScript/ES6

---

## Step 1: Project Setup

Create a new project directory and initialize it:

```bash
mkdir my-pict-app
cd my-pict-app
npm init -y
```

Install the required dependencies:

```bash
npm install pict pict-application pict-view
```

For browser-based development, also install build tools:

```bash
npm install --save-dev quackage
```

---

## Step 2: Create Your First Application

Create a file called `MyApplication.js`:

```javascript
const libPictApplication = require('pict-application');

const defaultConfiguration = {
    Name: 'My First Pict Application',
    MainViewportViewIdentifier: 'MainView',
    MainViewportDestinationAddress: '#app',
    AutoRenderMainViewportViewAfterInitialize: true
};

class MyApplication extends libPictApplication {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }

    onInitialize() {
        super.onInitialize();

        // Set up initial application data
        this.pict.AppData.Message = 'Hello, Pict!';
        this.pict.AppData.Items = [
            { id: 1, name: 'First Item' },
            { id: 2, name: 'Second Item' },
            { id: 3, name: 'Third Item' }
        ];

        this.log.info('Application initialized with data');
        return true;
    }
}

module.exports = MyApplication;
module.exports.default_configuration = defaultConfiguration;
```

---

## Step 3: Create a View

Create a file called `MyView.js`:

```javascript
const libPictView = require('pict-view');

const defaultViewConfiguration = {
    ViewIdentifier: 'MainView',
    DefaultRenderable: 'Main-Content',
    DefaultDestinationAddress: '#app',
    DefaultTemplateRecordAddress: 'AppData',

    AutoInitialize: true,
    AutoRender: true,

    Templates: [
        {
            Hash: 'Main-Template',
            Template: /*html*/`
                <div class="app-container">
                    <header>
                        <h1>{~Data:Record.Message~}</h1>
                    </header>
                    <main>
                        <h2>My Items</h2>
                        <ul id="item-list">
                            {~TS:Item-Row:AppData.Items~}
                        </ul>
                    </main>
                </div>
            `
        },
        {
            Hash: 'Item-Row',
            Template: /*html*/`
                <li data-id="{~Data:Record.id~}">{~Data:Record.name~}</li>
            `
        }
    ],

    Renderables: [
        {
            RenderableHash: 'Main-Content',
            TemplateHash: 'Main-Template',
            TemplateRecordAddress: 'AppData',
            DestinationAddress: '#app',
            RenderMethod: 'replace'
        }
    ]
};

class MyView extends libPictView {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }
}

module.exports = MyView;
module.exports.default_configuration = defaultViewConfiguration;
```

---

## Step 4: Create the Entry Point

Create `index.js`:

```javascript
const libPict = require('pict');
const MyApplication = require('./MyApplication');
const MyView = require('./MyView');

// Create Pict instance
const _Pict = new libPict({
    Product: 'My-Pict-App',
    ProductVersion: '1.0.0'
});

// Add the view first
_Pict.addView('MainView', MyView.default_configuration, MyView);

// Add and initialize the application
const myApp = _Pict.addApplication('MyApp', MyApplication.default_configuration, MyApplication);

// Initialize (this will auto-render due to configuration)
myApp.initialize();
```

---

## Step 5: Create the HTML Page

Create `index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>My Pict Application</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
            max-width: 800px;
            margin: 0 auto;
            padding: 20px;
        }
        .app-container {
            border: 1px solid #ddd;
            border-radius: 8px;
            padding: 20px;
        }
        header h1 {
            color: #333;
            border-bottom: 2px solid #007bff;
            padding-bottom: 10px;
        }
        ul {
            list-style: none;
            padding: 0;
        }
        li {
            padding: 10px;
            margin: 5px 0;
            background: #f5f5f5;
            border-radius: 4px;
        }
    </style>
</head>
<body>
    <div id="app">Loading...</div>

    <!-- Include your bundled JavaScript -->
    <script src="dist/my-app-bundle.js"></script>
</body>
</html>
```

---

## Step 6: Bundle for Browser

Create a Quackage configuration file `.quackage.json`:

```json
{
    "bundle": {
        "entry": "index.js",
        "output": "dist/my-app-bundle.js"
    }
}
```

Build the bundle:

```bash
npx quack build
```

---

## Step 7: Run Your Application

You can use any static file server to serve your application:

```bash
npx serve .
```

Open your browser to `http://localhost:3000` and you should see your application running!

---

## Next Steps

### Adding Interactivity

Add event handlers in your view:

```javascript
class MyView extends libPictView {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }

    onAfterRender() {
        super.onAfterRender();

        // Add click handlers
        const items = document.querySelectorAll('#item-list li');
        items.forEach(item => {
            item.addEventListener('click', () => {
                const id = item.dataset.id;
                this.log.info(`Clicked item ${id}`);
            });
        });
    }
}
```

### Adding a Data Provider

Create `MyDataProvider.js`:

```javascript
const libPictProvider = require('pict-provider');

class MyDataProvider extends libPictProvider {
    constructor(pFable, pOptions, pServiceHash) {
        super(pFable, pOptions, pServiceHash);
    }

    onLoadDataAsync(fCallback) {
        // Simulate API call
        setTimeout(() => {
            this.pict.AppData.Items = [
                { id: 1, name: 'Loaded Item 1' },
                { id: 2, name: 'Loaded Item 2' }
            ];
            this.log.info('Data loaded from provider');
            fCallback();
        }, 500);
    }
}

module.exports = MyDataProvider;
```

Register it in your entry point:

```javascript
_Pict.addProvider('DataProvider', {
    AutoInitialize: true,
    AutoLoadDataWithApp: true
}, MyDataProvider);
```

### Multiple Views

Add navigation between views by creating multiple view files and switching between them:

```javascript
// Switch to a different view
myApp.render('OtherView', 'Other-Content', '#app');
```

---

## Common Patterns

### Configuration-Only Views

For simple views, you don't need a custom class:

```javascript
_Pict.addView('SimpleView', {
    ViewIdentifier: 'SimpleView',
    Templates: [/* ... */],
    Renderables: [/* ... */]
});
```

### Async Initialization

For applications that need async setup:

```javascript
myApp.initializeAsync((error) => {
    if (error) {
        console.error('Initialization failed:', error);
        return;
    }
    console.log('Application ready!');
});
```

### Logging

Control log verbosity:

```javascript
_Pict.LogNoisiness = 3; // Higher = more verbose (0-5)
_Pict.LogControlFlow = true; // Log lifecycle events
```

---

## Troubleshooting

### Views Not Rendering

1. Check that the destination selector exists in the DOM
2. Verify `AutoRender: true` is set
3. Ensure views are added before the application initializes

### Data Not Appearing

1. Check the `TemplateRecordAddress` path is correct
2. Verify data exists in `AppData` before rendering
3. Use `_Pict.LogNoisiness = 5` to debug template processing

### Build Errors

1. Ensure all dependencies are installed
2. Check for circular dependencies
3. Verify file paths in require statements

---

## Resources

- [Pict Application GitHub](https://github.com/fable-retold/pict-application)
- [Pict View Documentation](https://github.com/fable-retold/pict-view)
- [Example Applications](https://github.com/fable-retold/pict-application/tree/main/example_applications)
