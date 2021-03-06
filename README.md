# SelfNetFS

A file system for web applications.

## Preparation

You may use the library in your plain old HTML page by `<script>` including the library
from unpkg (library is namespaced as `SNFS`):

```html
<script src="https://unpkg.com/selfnetfs/dist/selfnetfs.js"></script>
<!-- OR with a specific semantic-versioning version or range -->
<script src="https://unpkg.com/selfnetfs@^1.0.0/dist/selfnetfs.js"></script>
```

For web applications with build steps (e.g. React applications), you can install the npm
package:

```bash
npm install selfnetfs
```

And then import it:

```javascript
const SNFS = require('selfnetfs');
```

## Example

The following is a simplistic example of this library's usage that outlines authentication,
writing a file, and reading a file.

```javascript
let api = null;
let ses = null;
let fs = null;

async function startup() {
  api = new SNFS.Http('https://selfnetfs-api.home.arpa/myowner');
  try {
    ses = await api.login({
      name: 'guest',
      password: 'password',
    });
    fs = await ses.fs();
  } catch (err) {
    // Do something appropriate with the error.
    api = null;
    ses = null;
    fs = null;
  }
}

async function save() {
  const data = 'Data from some place. Note: this is a string.';
  try {
    // A directory does not need to be present to add files
    // to it. A string must be text-encoded first since the
    // library requires a Uint8Array.
    await fs.writefile('/path/to/my/file', new TextEncoder().encode(data));
  } catch (err) {
    // Do something appropriate with the error.
  }
}

async function load() {
  try {
    const result = await fs.readfile('/path/to/my/file');
    const data = new TextDecoder().decode(result.data);
    // Do something with the data.
  } catch (err) {
    // Do something appropriate with the error.
  }
}
```

An alternative startup logic that resumes a session from a previous page load
might look like this:

```javascript
let api = null;
let ses = null;
let fs = null;

async function startup() {
  const owner_url = localStorage.getItem('snfs_owner_url') || 'https://selfnetfs-api.home.arpa/myowner';
  const session_token = localStorage.getItem('snfs_session_token');
  api = new SNFS.Http(owner_url);
  try {
    ses = await api.resume(session_token);
    console.log('Successfully resumed session.');
  } catch (err) {
    console.log('Couldn\'t resume, logging in again.');
    ses = await api.login({ name: 'myuser', password: 'password' });
    localStorage.setItem('snfs_owner_url', owner_url);
    localStorage.setItem('snfs_session_token', ses.info().session_token);
  }
  fs = await ses.fs();
}
```

I encourage you to look at the source file [lib/snfs.ts](/lib/snfs.ts) for
a more complete listing of the methods and classes available when using the
library.

## Running the Shell/Tests

Because of the current state of browser security, testing this library
requires an involved setup (unless you already have stuff in place for HTTPS or
unless your browser considers http://127.0.0.1/ a secure host). The details for
how to do most steps should be familiar to you if you're familiar with running
web servers and familiar with NodeJS.

* Check out a copy of this repository.
* Install libraries with `npm install`.
* Build the browser library by running `npm run build`.
  This will produce the file `dist/selfnetfs.js` which you
  can `<script src="...">` include into your page.
* Bootstrap a new database by running the server with
  the `--init` flag:
  `ts-node server/index.ts --init ownername username password`.
  Be sure to clear you shell history (or `unset HISTFILE` if you're using bash)
  to ensure the password isn't in your history.
* Run the server with the command `npm run start` (listens on port 4000).
* Serve the example www assets with `npm run www` (listens on port 4001, requires `python3`).
* Have `selfnetfs-ui.home.arpa` point to `127.0.0.1` in `/etc/hosts`.
* Have `selfnetfs-api.home.arpa` point to `127.0.0.1` in `/etc/hosts`.
* Set up nginx to serve both `.home.arpa` addresses with a valid SSL certificate (I self
  signed mine and trusted it in my browser).
  - Reverse proxy `selfnetfs-api.home.arpa` to `127.0.0.1:4000`.
  - Reverse proxy `selfnetfs-ui.home.arpa` to `127.0.0.1:4001`.
* Navigate your browser to `https://selfnetfs-ui.home.arpa/shell.html` or
  `https://selfnetfs-ui.home.arpa/test.html`.
* In the shell, the command `help` lists the available commands.

## Notes

Without a session token private key, the server will use a randomly generated
key for the session JWT. To use a key that allows sessions to be resumed
between restarts of the server, a key can be created:

```
openssl ecparam -name secp256k1 -genkey -noout -out sestoken.pem
```
