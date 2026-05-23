# Pict Application

> **[&#9654; Read the Pict-Application Documentation](https://stevenvelozo.github.io/pict-application/)** &mdash; interactive docs with the full API reference.

Application base class for building pict view-based applications. Provides structured lifecycle management, view coordination, and data marshaling for web, console, and other applications where the UI is primarily represented as text strings.

## Installation

```bash
npm install pict-application
```

## Usage

```js
const libPict = require('pict');
const libPictApplication = require('pict-application');

// Create a Pict instance
const _Pict = new libPict();

// Add an application
const myApp = _Pict.addApplication('MyApp', {
    Name: 'My Application',
    MainViewportViewIdentifier: 'Main-View',
    AutoRenderMainViewportViewAfterInitialize: true
}, libPictApplication);

// Initialize the application
myApp.initialize();
```

## Documentation

- [README](docs/README.md) - Full documentation and API reference
- [Getting Started](docs/GETTING_STARTED.md) - Step-by-step tutorial
- [Configuration](docs/CONFIGURATION.md) - Complete configuration reference
- [Examples](docs/EXAMPLES.md) - Example application walkthroughs

## Related Projects

- [pict](https://github.com/stevenvelozo/pict) - Core Pict framework
- [pict-view](https://github.com/stevenvelozo/pict-view) - View base class
- [pict-provider](https://github.com/stevenvelozo/pict-provider) - Provider base class
- [fable](https://github.com/stevenvelozo/fable) - Service provider framework

## Related Packages

- [pict](https://github.com/stevenvelozo/pict) - MVC application framework
- [pict-view](https://github.com/stevenvelozo/pict-view) - View base class
- [pict-provider](https://github.com/stevenvelozo/pict-provider) - Data provider base class
- [fable](https://github.com/stevenvelozo/fable) - Application services framework

## License

MIT

## Contributing

Pull requests are welcome. For details on our code of conduct, contribution process, and testing requirements, see the [Retold Contributing Guide](https://github.com/stevenvelozo/retold/blob/main/docs/contributing.md).
