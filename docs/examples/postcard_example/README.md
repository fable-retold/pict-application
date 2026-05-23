# Postkard — Bootstrapping a Pict Application

<!-- docuserve:example-launch:start -->
> **[&#9654; Launch the live app](examples/postcard%5Fexample/index.html)** — runs in your browser, opens in a new tab.
<!-- docuserve:example-launch:end -->

The Postkard example is the canonical "look how little you need" demo for
`pict-application`. It is a complete browser application — a postcard
authoring flow with a hand-drawn navigation, two static content pages, a
custom Pure-CSS form theme, and a "mock server" provider that swaps form
configuration in at startup — and the entire wiring fits in a single
**`Pict-Application-Postcard.js`** file.

This page walks the example end-to-end so the patterns generalize. Read
the bootstrap, then each `Feature N` deep-dive; by the end you should
recognize how every `pict-application` host you ever write splits the
same way: **constructor** wires providers and views, **lifecycle hooks**
(`onAfterInitializeAsync`, `onLoginAsync`, `onLoadDataAsync`) coordinate
the application's startup, and **navigation handlers** call `render()`
on the appropriate view.

## What it demonstrates

| Capability | Where you see it |
|------------|------------------|
| Extending `PictApplication` for a real app | `class PostcardApplication extends libPictApplication` in `Pict-Application-Postcard.js` |
| Registering providers in the constructor | `this.pict.addProvider('Postcard-DynamicSection-Provider', …)` for the mock data provider, plus a second provider for the theme |
| Registering services and views | `this.pict.addServiceType('PictSectionForm', …)`, `this.pict.addView('PictFormMetacontroller', …)`, `this.pict.addView('PostcardMainApplication', …)` |
| `ConfigurationOnlyViews` — views with no custom class | The Navigation, About, and Legal pages live as pure JSON, loaded via `ConfigurationOnlyViews: [ require(…), require(…), require(…) ]` |
| `AutoLoginAfterInitialize` + `AutoLoadDataAfterLogin` lifecycle | `loggedIn` flag, `isLoggedIn()`, `onLoginAsync()`, `onLoadDataAsync()` — the framework chains them automatically |
| `onInitializeAsync` provider hook for "fetched" config | The Dynamic Sections provider plants `pict.settings.DefaultFormManifest` from a "mock server response" before the form metacontroller initializes |
| Pluggable form template theme | A second provider (`Postcard-Default-Theme-Provider`) registers a complete PureCSS `<form class="pure-form">` template set; the navigation can swap themes at runtime |
| `onAfterInitializeAsync` to wire up the first render | Pick a default theme, set `viewMarshalDestination`, render Navigation + Main + FormMetacontroller |
| `marshalDataFromViewToAppData()` / `marshalDataFromAppDataToView()` round-trip | The "Store Data" and "Read Data" nav links call these directly |
| Multi-view navigation via inline `onclick` handlers | The Navigation template's `<a onclick="_Pict.views.PostcardAbout.render()">` links — no router needed |

## Key files

- `Pict-Application-Postcard.js` — the class. Constructor registers the two
  providers, the form service, the metacontroller view, and the main
  application view. The `default_configuration` block at the bottom lists
  the three `ConfigurationOnlyViews` (Navigation, About, Legal) and sets
  `MainViewportViewIdentifier: "PostcardNavigation"`. Read it
  top-to-bottom — the pattern every `pict-application` host follows.
- `providers/PictProvider-Dynamic-Sections.js` — a `pict-provider`
  subclass whose `onInitializeAsync` plants a `DefaultFormManifest` onto
  `pict.settings`. Acts as a stand-in for "fetch the form config from
  the server, then continue boot."
- `providers/PictProvider-Dynamic-Sections-MockServerResponse.json` —
  the form manifest the dynamic-section provider injects. Two sections,
  six groups, a dozen descriptors driving Recipient, Message,
  Delivery Destination, and Sender Contact data.
- `providers/PictProvider-BestPostcardTheme.js` — extends
  `pict-section-form`'s `PictFormTemplateProvider`. Ships a complete
  `_Theme` block (`TemplatePrefix`, `Templates: [...]`) that styles
  every form fragment (`Wrap`, `Section`, `Group`, `Row`, `Input`) as
  PureCSS markup.
- `views/PictView-Postcard-MainApplication.js` — the only view with a
  custom class. Its `onAfterRender()` re-runs the form metacontroller
  and marshals data back to the DOM.
- `views/PictView-Postcard-Navigation.json` — a configuration-only view.
  No JS class — just `Templates: [...]` + `Renderables: [...]`, loaded
  via `ConfigurationOnlyViews` in the app config.
- `views/PictView-Postcard-Content-About.json` /
  `PictView-Postcard-Content-Legal.json` — same shape: pure JSON
  configuration-only views with embedded HTML content.
- `html/index.html` — the shell. Loads `pict.min.js`, calls
  `Pict.safeLoadPictApplication(PostcardApplication, 2)` once the DOM
  is ready, and provides two destination divs the views render into
  (`#Postcard-Navigation-Container`, `#Postcard-Content-Container`).

## The data model

`AppData.PostKard` holds the entire postcard payload — set as the
metacontroller's `viewMarshalDestination` so every input writes back to
that one root. The form manifest seeds it from the "server response":

- **Postcard Information section** — `RecipientName`, `SendDate`,
  `MessageTitle`, `MessageBody`, `SignatureLine`, and a
  `Destination` group with `StreetAddress1/2`, `City`, `State`, `Zip`.
- **Confirmation section** — `SenderEmailAddress`, `SenderPhoneNumber`,
  `SenderExplicitMarketingConsent` (a `Boolean`).

Each descriptor's `PictForm` block names the section, group, row and
width — `pict-section-form` reads those and lays out the inputs. No
view code touches this; the metacontroller materializes the entire form
from configuration.

---

## Feature 1 — Extending `PictApplication`

The shape every `pict-application` host shares: a class extends
`libPictApplication`, the constructor registers providers, services and
views, and a `default_configuration` block at module scope names the
main viewport and the configuration-only views.

```javascript
const libPictApplication = require('../../source/Pict-Application.js');

const libPictSectionForm = require('pict-section-form');

const libProviderDynamicSection = require('./providers/PictProvider-Dynamic-Sections.js');

const libMainApplicationView = require('./views/PictView-Postcard-MainApplication.js')

class PostcardApplication extends libPictApplication
{
	constructor(pFable, pOptions, pServiceHash)
	{
		super(pFable, pOptions, pServiceHash);

		this.pict.addProvider('Postcard-DynamicSection-Provider', libProviderDynamicSection.default_configuration, libProviderDynamicSection);
		this.pict.addProvider('Postcard-Default-Theme-Provider', {}, require('./providers/PictProvider-BestPostcardTheme.js'));

		// Add the pict form service
		this.pict.addServiceType('PictSectionForm', libPictSectionForm);

		// Add the pict form metacontroller service, which provides programmaatic view construction from manifests and render/marshal methods.
		this.pict.addView('PictFormMetacontroller', {}, libPictSectionForm.PictFormMetacontroller);

		this.pict.addView('PostcardMainApplication', libMainApplicationView.default_configuration, libMainApplicationView);

		this.loggedIn = false;
	}
	// …
}
```

The pattern: `super()` first, then a series of `this.pict.addProvider(…)`
/ `addServiceType(…)` / `addView(…)` calls. The order matters only when
one registration's `AutoInitializeOrdinal` ranks higher than another's
— for this app order doesn't matter because the metacontroller defers
its work until `onAfterInitializeAsync` (see Feature 4).

## Feature 2 — `ConfigurationOnlyViews` — views with no class

Not every view needs a JS class. If all a view does is map a template
to a destination and re-render on click, JSON is enough. The base class
reads `options.ConfigurationOnlyViews` during `initialize()` and adds
each entry via `this.pict.addView()` — the same call you'd make
manually.

```javascript
module.exports.default_configuration = (
{
	"Name": "A Simple Postcard Application",
	"Hash": "Postcard",

	"MainViewportViewIdentifier": "PostcardNavigation",
	"AutoLoginAfterInitialize": true,
	"AutoLoadDataAfterLogin": true,

	"ConfigurationOnlyViews":
	[	require('./views/PictView-Postcard-Navigation.json'),
		require('./views/PictView-Postcard-Content-About.json'),
		require('./views/PictView-Postcard-Content-Legal.json')
	],

	"pict_configuration":
		{
			"Product": "Postkard-Pict-Application"
		}
});
```

A configuration-only view file is just the same JSON object you'd pass
to `addView()`:

```json
{
    "ViewIdentifier": "PostcardAbout",
    "DefaultRenderable": "Postcard-About",
    "DefaultDestinationAddress": "#Postcard-Content-Container",
    "AutoRender": false,
    "Templates": [
        { "Hash": "Postcard-About-Content", "Template": "…" }
    ],
    "Renderables": [
        { "RenderableHash": "Postcard-About", "TemplateHash": "Postcard-About-Content" }
    ]
}
```

That's the entire **About** page. No class file. The Legal and
Navigation pages have the identical shape. Use this pattern any time a
view is "render this template to this destination on demand" — reach
for a custom class only when you need `onBeforeRender` / `onAfterRender`
hooks to coordinate other behaviour, as the Main Application view does.

## Feature 3 — Provider that mocks a fetched form manifest

The Dynamic-Sections provider stands in for "we fetched the form
config from the server on startup." It extends `pict-provider`,
declares `AutoInitialize: true`, and overrides `onInitializeAsync` to
plant the mock manifest on `pict.settings.DefaultFormManifest` — which
the form metacontroller reads on its own initialization.

```javascript
const libPictProvider = require('pict-provider');

const _DEFAULT_PROVIDER_CONFIGURATION =
{
	ProviderIdentifier: 'Postcard-DynamicSection-Provider',

	AutoInitialize: true,
	AutoInitializeOrdinal: 0,
}

class PostcardDynamicSectionProvider extends libPictProvider
{
	constructor(pFable, pOptions, pServiceHash)
	{
		let tmpOptions = Object.assign({}, _DEFAULT_PROVIDER_CONFIGURATION, pOptions);
		super(pFable, tmpOptions, pServiceHash);
	}

	onLoadDataAsync(fCallback)
	{
		setTimeout(() =>
		{
			this.log.info('PostcardDynamicSectionProvider.onLoadDataAsync() called --- simulating data load from "server".');
			fCallback();
		}, 100);
	}

	onInitializeAsync(fCallback)
	{
		this.log.info('PostcardDynamicSectionProvider.onInitializeAsync() called --- loading dynamic section views from "server".');
		// Load the dynamnic section views from the server
		this.pict.settings.DefaultFormManifest = require('./PictProvider-Dynamic-Sections-MockServerResponse.json');
		this.log.info('PostcardDynamicSectionProvider.onInitializeAsync() -- loaded dynamic section views from "server" to the application [settings.DefaultFormManifest] location.');

		return super.onInitializeAsync(fCallback);
	}
}
```

`PictApplication.initializeAsync()` walks every provider with
`AutoInitialize: true`, sorted by `AutoInitializeOrdinal`, and
`anticipate`s their `initializeAsync()` calls in order. So
`pict.settings.DefaultFormManifest` is populated **before** the
metacontroller view's own `initializeAsync` fires and reads it. In a
real application you would replace the synchronous `require(…)` with
an HTTP call inside the `setTimeout` — the rest of the boot sequence
keeps working without change.

## Feature 4 — `onAfterInitializeAsync` chains the first render

Because the form metacontroller's `AutoRender` is off (the
metacontroller needs its host views to exist first), the application
hooks `onAfterInitializeAsync` to walk through the wake-up sequence:

```javascript
onAfterInitializeAsync(fCallback)
{
	// Default to the Pure theme
	this.pict.views.PictFormMetacontroller.formTemplatePrefix = _Pict.providers['Postcard-Default-Theme-Provider'].formsTemplateSetPrefix;

	// Set a custom address for all the views to marshal to.
	// This can also be set on specific views (same property)
	this.pict.views.PictFormMetacontroller.viewMarshalDestination = 'AppData.PostKard';

	this.pict.views.PostcardNavigation.render()
	this.pict.views.PostcardMainApplication.render();
	this.pict.views.PictFormMetacontroller.render();

	return super.onAfterInitializeAsync(fCallback);
}
```

What's happening here, line by line:

1. **Pick a default theme.** Read the postcard theme provider's
   `formsTemplateSetPrefix` (just the string `"Postcard-Theme"`) and
   give it to the metacontroller so it uses that template set when
   it materializes the form.
2. **Pin every input's marshal destination** to
   `AppData.PostKard.<descriptor.Hash>`. Without this each descriptor
   would marshal to its own per-descriptor address — the single root
   makes the saved postcard a single object the host can
   `JSON.stringify` and ship to a server.
3. **Render the navigation** — populates
   `#Postcard-Navigation-Container` with the menu.
4. **Render the main application view** — its template writes
   `<h1>Send a Fabulous Postkard!</h1>` plus a `<div
   id="Pict-Form-Container">` into `#Postcard-Content-Container`.
5. **Render the form metacontroller** — fills the form container with
   sections/groups/inputs derived from the manifest.

The `super.onAfterInitializeAsync(fCallback)` is non-negotiable — it
lets the base class trigger `AutoLoginAfterInitialize` →
`onLoginAsync` → (because `AutoLoadDataAfterLogin` is set)
`onLoadDataAsync` in order.

## Feature 5 — Login + LoadData lifecycle hooks

The framework provides an explicit, callback-driven login/load-data
pipeline so apps can decide what "logged in" means without rewriting
the boot order. Postkard simulates a 100ms network round-trip for
each:

```javascript
onLoginAsync(fCallback)
{
	// simulate a check session
	setTimeout(() =>
	{
		this.loggedIn = true;
		this.log.info('PostcardApplication: onLoginAsync: Simulating login...');
		return super.onLoginAsync(fCallback);
	}, 100);
}

isLoggedIn()
{
	return this.loggedIn;
}

onLoadDataAsync(fCallback)
{
	// simulate a load data
	setTimeout(() =>
	{
		this.data = { };
		this.log.info('PostcardApplication: onLoadDataAsync: Simulating data load...');
		return super.onLoadDataAsync(fCallback);
	}, 100);
}
```

`AutoLoginAfterInitialize: true` (in the app config) tells the base
class to call `loginAsync` after `onAfterInitializeAsync`. The base
class's `loginAsync` runs `onBeforeLoginAsync` → `onLoginAsync` →
`onAfterLoginAsync`, then — because `AutoLoadDataAfterLogin: true` —
gates on `isLoggedIn()` returning truthy and runs
`loadDataAsync` (`onBeforeLoadDataAsync` → `onLoadDataAsync` →
`onAfterLoadDataAsync`, also walking every provider with
`AutoLoadDataWithApp`). Each hook is a place to wedge the host's own
async work without rewriting the orchestration. Override
`isLoggedIn()` to return `false` and the framework will short-circuit
the data load — useful for a "you must sign in" screen.

## Feature 6 — Pluggable form-theme provider

The Postcard theme is its own `pict-section-form` template provider —
a complete set of fragment templates (`Wrap-Prefix`, `Wrap-Postfix`,
`Section-Prefix`, `Section-Postfix`, `Group-Prefix`, `Row-Prefix`,
`Input`, `Input-DataType-Number`, `Input-InputType-TextArea`, …) that
collectively render every form fragment as PureCSS markup.

```javascript
const libPictFormSection = require('pict-section-form');

const _Theme = (
{
	"TemplatePrefix": "Postcard-Theme",

	"Templates":
		[
			{
				"HashPostfix": "-Template-Section-Prefix",
				"Template": /*HTML*/`
			<!-- Form Section Prefix [{~D:Context[0].UUID~}]::[{~D:Context[0].Hash~}] {~D:Record.Hash~}::{~D:Record.Name~} -->
<form class="pure-form pure-form-stacked">
    <fieldset>
	`
			},
			// … wrap / group / row / input templates …
			{
				"HashPostfix": "-Template-Input-DataType-Number",
				"Template": /*HTML*/`
            <div class="pure-u-1 pure-u-md-1-3">
                <label {~D:Record.Macro.HTMLForID~}>{~D:Record.Name~}:</label>
                <input type="number" {~D:Record.Macro.HTMLID~} {~D:Record.Macro.InputFullProperties~} class="pure-u-23-24" />
            </div>
	`
			}
		]
});

class PostcardTheme extends libPictFormSection.PictFormTemplateProvider
{
	constructor(pFable, pOptions, pServiceHash)
	{
		let tmpOptions = Object.assign({}, pOptions, {MetaTemplateSet:_Theme});
		super(pFable, tmpOptions, pServiceHash);
	}
}
```

`PictFormTemplateProvider` registers each fragment as
`<TemplatePrefix><HashPostfix>` — `Postcard-Theme-Template-Section-Prefix`,
`Postcard-Theme-Template-Input-DataType-Number`, etc. — and exposes
`formsTemplateSetPrefix` (the `"Postcard-Theme"` value). Telling the
metacontroller to use a different prefix is enough to swap the theme,
which is exactly how the Navigation's "Postkard Pure CSS Theme" link
works:

```javascript
changeToPostcardTheme()
{
	this.pict.views.PictFormMetacontroller.formTemplatePrefix = _Pict.providers['Postcard-Default-Theme-Provider'].formsTemplateSetPrefix;
	this.pict.views.PictFormMetacontroller.regenerateFormSectionTemplates();
	this.pict.views.PictFormMetacontroller.renderFormSections();
	this.marshalDataFromAppDataToView();
}

changeToDefaultTheme()
{
	this.pict.views.PictFormMetacontroller.formTemplatePrefix = _Pict.providers.PictFormSectionDefaultTemplateProvider.formsTemplateSetPrefix
	this.pict.views.PictFormMetacontroller.regenerateFormSectionTemplates();
	this.pict.views.PictFormMetacontroller.renderFormSections();
	this.marshalDataFromAppDataToView();
}
```

Three calls: change the prefix, regenerate the per-section templates
from the new prefix, re-render the form. The final `marshalDataFromAppDataToView()`
pushes the current `AppData.PostKard` payload back into the new HTML
so the user's in-progress postcard isn't lost across a theme swap.

## Feature 7 — Multi-view navigation via inline `onclick`

There is no router. Each nav link is an inline `onclick="..."` that
calls `_Pict.views.<X>.render()` (or an app method) directly. The
navigation template lives inside the configuration-only Navigation
view:

```html
<span class="pure-menu-heading">Postkard</span>
<ul class="pure-menu-list">
    <li class="pure-menu-item"><a href="#" onclick="_Pict.views.PostcardMainApplication.render()" class="pure-menu-link">Send a Kard</a></li>
    <li class="pure-menu-item"><a href="#" onclick="_Pict.views.PostcardAbout.render()" class="pure-menu-link">About</a></li>
    <li class="pure-menu-item"><a href="#" onclick="_Pict.views.PostcardLegal.render()" class="pure-menu-link">Legal</a></li>
    <li class="pure-menu-item menu-item-divided"><a href="#" onclick="_Pict.PictApplication.marshalDataFromViewToAppData()" class="pure-menu-link">Store Data</a></li>
    <li class="pure-menu-item"><a href="#" onclick="_Pict.PictApplication.marshalDataFromAppDataToView()" class="pure-menu-link">Read Data</a></li>
    <li class="pure-menu-item menu-item-divided"><a href="#" onclick="_Pict.PictApplication.changeToPostcardTheme()" class="pure-menu-link">Postkard Pure CSS Theme</a></li>
    <li class="pure-menu-item"><a href="#" onclick="_Pict.PictApplication.changeToDefaultTheme()" class="pure-menu-link">Pict Raw HTML Form Theme</a></li>
</ul>
```

All four content destinations target the same
`#Postcard-Content-Container` div, so each `render()` replaces the
previous content. Because all three content views have `AutoRender:
false`, the only way they appear is by being rendered explicitly —
either by the app at startup (Main) or by the user clicking a nav
link.

The bottom of the menu also exposes the two data-marshal entry
points: **Store Data** (`marshalDataFromViewToAppData()` — pull the
DOM state back into `AppData.PostKard`) and **Read Data**
(`marshalDataFromAppDataToView()` — push `AppData.PostKard` back into
the DOM). Type a recipient name, hit Store Data, switch to About and
back, hit Read Data — the recipient name is still there. That's the
visible proof that `AppData.PostKard` is the source of truth, not the
DOM.

For a real application the playbook recommendation is to swap this
ad-hoc menu for `pict-router` — every nav link becomes a route, every
`onclick` becomes a `navigateTo('/path')`, and browser back/forward
starts working. See the `pict-router` example for the upgrade path.

## Running the example

```bash
cd example_applications/postcard_example
npm install        # installs pict-section-form
npm run build      # quack build + quack copy → ./dist
# then open dist/index.html in a browser
```

The example uses CDN-loaded Pure CSS for layout, so the only locally
served file is the application bundle.

## Things to try in the running app

- **Type into the recipient name field**, then click **Store Data**.
  Open the browser console and inspect
  `_Pict.AppData.PostKard.RecipientName` — your value is there.
- **Click About**, then **Legal**, then **Send a Kard**. The content
  pane swaps; the form re-renders from `AppData.PostKard` (any saved
  state stays).
- **Click "Pict Raw HTML Form Theme"**. The form re-renders without
  PureCSS styling — same data, completely different markup. Click
  "Postkard Pure CSS Theme" to flip back.
- **Reload the page**. Without explicit persistence, the postcard
  resets — the framework intentionally doesn't assume localStorage. A
  real app would wire `onSaveDataAsync` to write to the server or
  localStorage at the bottom of the navigation.
- **Watch the console.** With `LogNoisiness` cranked up you can see
  every `onBeforeInitialize`, `onInitialize`, `onAfterInitialize`,
  `onLoginAsync`, and `onLoadDataAsync` callback fire in order — the
  full lifecycle is visible.

## Takeaways

1. **The constructor is the wiring diagram.** Every provider, service
   and view a host needs is registered in the constructor; there is
   never any "register at first render" cleverness. Read the
   constructor and you've read the architecture.
2. **`ConfigurationOnlyViews` is the right answer for static
   content.** Don't write a custom `pict-view` class for "render this
   template to this destination." Wire it as JSON in the app config
   and let the framework do the rest.
3. **Providers do the boot work, lifecycle hooks chain the
   sequence.** The dynamic-section provider plants the manifest in
   `onInitializeAsync`; the application stitches "render main view +
   form" in `onAfterInitializeAsync`; login and data-load fall out of
   the framework's automatic chaining once `AutoLoginAfterInitialize`
   / `AutoLoadDataAfterLogin` are set.
4. **Form themes are providers.** A template-prefix swap is enough to
   re-skin every form input. Keeping a theme as a provider (not a
   hardcoded template set) means a host can ship multiple themes and
   pick one at runtime, exactly as the postcard does.
5. **`AppData` is the application's source of truth.** The marshal
   methods round-trip between DOM and `AppData`; the views render
   from `AppData`. A theme swap, a navigation, a save — none of them
   touch the DOM directly, and that's why `AppData.PostKard` survives
   all of them.

## Related documentation

- [Getting Started](../../GETTING_STARTED.md) — minimum-viable
  `pict-application` host
- [Configuration](../../CONFIGURATION.md) — the full app /view /
  provider configuration reference, including `AutoLoginAfterInitialize`,
  `AutoLoadDataAfterLogin`, `ConfigurationOnlyViews`, and the auto
  ordinals
- [Examples Overview](../../EXAMPLES.md) — companion walk-throughs of
  the other example_applications shipped with this module
