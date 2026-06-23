QUESTION

A customer asked whether **node_modules / npm dependencies** can be used inside **Launch cloud functions**, and if so: what runtime is supported, how builds/dependencies should be structured, and what the **caching behavior** is for cloud function responses.

ANSWER

**Runtime support**

- Launch cloud functions currently run on the **Node.js 16** runtime.

**Using dependencies (node_modules)**

- Yes, **node_modules and npm dependencies are supported** in Launch cloud functions.
- Dependencies declared in your project's **`package.json`** are available to cloud functions in the normal way—no special configuration is required beyond what any JavaScript project needs.
- The `package.json` file lives at the **project root** (standard JavaScript project layout). Imports inside function code work as usual.

**Building dependencies**

- Any build-generation steps (e.g. `npm install`, bundling, compilation) are handled via the **Build command** option in Launch's project settings. Set your build command there; Launch will execute it before deploying.

**Caching behavior**

- By default, cloud function responses are set to **`no-cache`**.
- To enable caching, set **`Cache-Control` response headers** inside your function code. Launch will honour whatever cache directives you return, giving you full control over cache behaviour (e.g. `Cache-Control: public, max-age=60`).

**Suggested documentation additions** (raised in thread)

- An example repository / file-structure showing `package.json` at root alongside a `/functions` directory.
- A sample import statement within a cloud function using an installed dependency.
- A note in the cloud functions docs clarifying that no custom build command is required solely to enable `package.json` dependencies; the standard project build command suffices.
