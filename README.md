# tally-extension

Tally is a community owned and operated Web3 wallet, built as a
[WebExtension](https://browserext.github.io/browserext/).

## Package Structure, Build Structure, and Threat Model

The extension is built as two packages, one for the wallet and one for the
frontend UI. These are separate packages in order to emphasize the difference
in attack surface and clearly separate the threat models of the two packages.
In particular, the frontend UI is considered completely untrusted code, while
the wallet is considered trusted code. Only the wallet should interact directly
with key material, while the frontend should only interact with key material
via a carefully-maintained public API.

The wallet package is also intended to minimize external dependencies where
possible, to reduce the surface exposed to a supply chain attack. Dependencies
are generally version-pinned, and yarn is used to ensure the integrity of
builds.

## Building and Developing

Builds are designed to be run from the top level of the repository.

### Quickstart

```sh
$ nvm use
$ yarn install # install all dependencies; rerun with --ignore-scripts if
               # scrypt node-gyp failures prevent the install from completing
$ yarn start # start a continuous webpack build that will auto-update with changes
```

Once the continuous webpack build is running, you can install the extension in
your dev browser of choice:

- [Firefox instructions](https://extensionworkshop.com/documentation/develop/temporary-installation-in-firefox/)
- [Chrome, Brave, and Opera instructions](https://developer.chrome.com/docs/extensions/mv3/getstarted/#manifest)
  - Note that these instructions are for Chrome, but substituting
    `brave://extensions` or `opera://extensions` for `chrome://extensions`
    depending on browser should get you to the same buttons.

Extension bundles for each browser are in `dist/<browser>`.

### Additional Scripts

```sh
$ yarn build # create a production build of the extension
```

The build script will generate a ZIP file for each browser bundle under the
`dist/` directory.

```sh
$ yarn lint # lint all sources in all projects
$ yarn lint-fix # auto-fix any auto-fixable lint issues
$ yarn test # run all tests in all projects

```

#### A note on `git blame`

Because lint configurations can occasionally evolve in a way that hits many
files in the repository at once and obscures the functional blame readout for
files, this repository has a `.git-blame-ignore-revs` file. This file can be
used to run `git blame` while skipping over the revisions it lists, as
described in [the Pro Git book
reference](https://www.git-scm.com/docs/git-blame#Documentation/git-blame.txt---ignore-revs-fileltfilegt)
and [this Moxio blog
post](https://www.moxio.com/blog/43/ignoring-bulk-change-commits-with-git-blame).

To make use of this, you can do one of the following:

- Run `git config --global blame.ignoreRevsFile .git-blame-ignore-revs` to
  configure git to globally look for such a file. The filename is relatively
  standard across projects, so this should save time for other projects that
  use a similar setup.
- Run `git config blame.ignoreRevsFile .git-blame-ignore-revs` to configure
  your local checkout to always ignore these files.
- Add `--ignore-revs-file .git-blame-ignore-revs` to your `git blame`
  invocation to ignore the file one time.

The GitHub UI does not yet ignore these commits, though there is a
[community thread requesting the
feature](https://github.community/t/support-ignore-revs-file-in-githubs-blame-view/3256).
In the meantime, the GitHub blame UI does allow you to zoom to the previous
round of changes on a given line, which relieves much of the annoyance; see
[the GitHub blame docs for
more](https://docs.github.com/en/github/managing-files-in-a-repository/managing-files-on-github/tracking-changes-in-a-file).

## File Structure

Extension content lives directly under the root directory alongside
project-level configuration and utilities, including GitHub-specific
functionality in `.github`. Extension content should be minimal, and
largely simply glue together UI and wallet code. Manifest information
is managed in the `manifest/` subdirectory as described below.

Here is a light guide to the directory structure:

```
.github/ # GitHub-specific tooling

package.json      # private extension package
webpack.config.js # Webpack build for extension

src/ # extension source files
  background.js # entry file for the background extension script; should be
                # minimal and call in to @tallyho/tally-wallet
  ui.js         # entry file for the frontend UI; should be minimal and bind
                # the functionality in @tallyho/tally-ui

dist/ # output directory for builds
  brave/   # browser-specific
  firefox/ # build
  chrome/  # directories
  brave.zip   # browser-specific
  firefox.zip # production
  chrome.zip  # bundles

build-utils/ # build-related helpers, used in webpack.config.js
  *.js
dev-utils/          # dev-mode helpers for the extension
  extension-reload.js # LiveReload support for the extension.
manifest/         # extension manifest data
  manifest.json             # common manifest data for all browsers
  manifest.chrome.json      # manifest adjustments for Chrome
  manifest.dev.json         # manifest adjustments for dev environment
  manifest.firefox.dev.json # manifest adjustments for Firefox in dev

ui/ # @tallyho/tally-ui package
  package.json

wallet/ # @tallyho/tally-wallet package with trusted wallet core
  package.json
```
