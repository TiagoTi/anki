# Local Sync Server (Self-Hosted)

> **Note:** These instructions apply to the bundled sync server in Anki 2.1.43. Newer versions of Anki (2.1.57+) use a different sync server implementation with different configuration options (such as `SYNC_USER1` and media sync support).

A local sync server is bundled with Anki. If you cannot or do not wish to
use AnkiWeb, you can run the server on a machine on your local network.

## Important Limitations

Please be aware of the following before using the local sync server:

- **Media Syncing:** Not currently supported in this version. You will need to either disable syncing of sounds and images in the preferences screen, sync your media via AnkiWeb, or use an alternative solution.
- **Client Support:** AnkiMobile does not yet provide an option for using a local sync server. This is currently only usable with the desktop version of Anki and AnkiDroid.
- **Security:** The server runs over an unencrypted HTTP connection and **does not require authentication**. It is only suitable for use on a private, trusted network.
- **Advanced Feature:** This is targeted at users comfortable with networking and the command line. You are expected to resolve any setup, network, or firewall issues yourself. Use is entirely at your own risk.

## Running from Source

If you are running Anki from a source checkout, you can start the sync server with:

```bash
./scripts/runopt --syncserver
```

This command uses Bazel to build and run the server.

## From a packaged build

From 2.1.39beta1+, the sync server is included in the packaged binaries.

On Windows in a cmd.exe session:

```
"\program files\anki\anki-console.exe" --syncserver
```

Or MacOS, in Terminal.app:

```
/Applications/Anki.app/Contents/MacOS/AnkiMac --syncserver
```

Or Linux:

```
anki --syncserver
```

## Without Qt Dependencies

You can run the server without installing the GUI portion of Anki. Once Anki
2.1.39 is released, the following will work:

```bash
pip install anki[syncserver]
python -m anki.syncserver
```

## Building a Standalone Wheel

If you want to build a redistributable Python wheel from source, you can use the provided build scripts.

On Linux or macOS:
```bash
./scripts/build
```

On Windows:
```bash
.\scripts\build.bat
```

The generated `.whl` files will be located in the `bazel-dist/` directory. You can then install the `anki` wheel using `pip`:

```bash
pip install bazel-dist/anki-*.whl[syncserver]
```

## Server Configuration

The following environment variables can be used to configure the server:

- `FOLDER`: The directory where the server will store collections. Defaults to `~/.syncserver`. **This must not be the same as your normal Anki data folder.**
- `HOST`: The IP address to bind to. Defaults to `0.0.0.0` (all interfaces).
- `PORT`: The port to listen on. Defaults to `8080`.

Example:
```bash
FOLDER=~/anki-server-data HOST=127.0.0.1 PORT=9000 anki --syncserver
```

## Client Configuration

To tell your Anki clients to use your local sync server, you need to set the following environment variables before starting the client:

- `SYNC_ENDPOINT`: The URL of your sync server's collection sync endpoint (e.g., `http://10.0.0.5:8080/sync/`).
- `SYNC_ENDPOINT_MEDIA`: (Optional) The URL for media syncing. Note that the bundled server does not currently support media sync, so this is typically left pointed at AnkiWeb or unset.

### Example (Linux/macOS)
```bash
export SYNC_ENDPOINT="http://10.0.0.5:8080/sync/"
anki
```

### Example (Windows)
```cmd
set SYNC_ENDPOINT=http://10.0.0.5:8080/sync/
anki-console.exe
```

Currently, any username and password will be accepted by the server. If you wish to keep using AnkiWeb for media, sync once with AnkiWeb first, then switch to your local endpoint. Collection syncs will be local, and media syncs will continue to go to AnkiWeb.

## Contributing

Authentication shouldn't be too hard to add - login() and request() in
http_client.rs can be used as a reference. A PR that accepts a password in an
env var, and generates a stable hkey based on it would be welcome.

Once that is done, basic multi-profile support could be implemented by moving
the col object into an array or dict, and fetching the relevant collection based
on the user's authentication.

Because this server is bundled with Anki, simplicity is a design goal - it is
targeted at individual/family use, only makes use of Python libraries the GUI is
already using, and does not require a configuration file. PRs that deviate from
this are less likely to be merged, so please consider reaching out first if you
are thinking of starting work on a larger change.
