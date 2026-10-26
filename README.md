The manifest builds Picocrypt NG with pinned Go 1.27.3 and offline vendored
dependencies. Fyne 2.8.2 supplies native Wayland and X11 in the same binary.
`PICOCRYPT_BACKEND=x11|wayland` selects a backend; the default `auto` keeps Fyne's
runtime detection and its X11 fallback for compositors without server-side
decorations. Both display sockets are intentional: `fallback-x11` would hide
X11 in a Wayland session and disable that upstream workaround for window resizing
crashes/hangs ([Fyne #6392](https://github.com/fyne-io/fyne/pull/6392)).

The `flatpak` build tag enables Fyne's URL and file-dialog portals. Filesystem
home access is retained for the application's file, folder, drag-and-drop, and
CLI workflows.

Read-only `/sys/fs/cgroup` access lets the existing PCV3 resource check account
for the process's effective memory limits. Without those kernel facts it refuses
encryption and decryption before creating output. The permission does not allow
the app to change cgroup limits or membership.

The source pins unreleased local candidate commit
`35b5824c68ebc2e5734e26c716e601d02ae2299e`. This commit is not yet available from
the public repository. Remote source retrieval and official Flathub acceptance
remain release gates. The launcher comes directly from this source; it needs no
packaging patch.

For local validation, export that exact tracked commit with `git archive`, record
the archive's SHA-256, and replace only the application's Git source in a separate
local manifest with an archive source pointing to that file. Keep the recipe's
other sources and build commands unchanged. After all sources are cached, build
with `flatpak-builder --disable-download` to verify offline availability. Do not
treat a local source override as proof that the public Git source can be fetched.

Before submitting a release update, select the reviewed public release commit,
refresh the source pin and vendored dependencies together, and validate both
supported architectures and the complete exported application.

For exported AppStream validation, build with `--compose-url-policy=full` and
`--mirror-screenshots-url=https://dl.flathub.org/media`. The builder downloads the
screenshot, rewrites the catalog URL, and exports a local screenshot ref. This
does not publish the screenshot. Screenshot composition may need network access
even when the application sources are already cached.

Local validation on 2026-10-26 with `flatpak-builder-lint` version
`3.0.0.post798.dev0+5181352` reports
`finish-args-contains-both-x11-and-wayland` and
`finish-args-home-filesystem-access`. The permissions retain the existing
backend fallback and file workflows described above. The linter also recommends
Freedesktop 26.08; this preparation keeps the existing 25.08 runtime.
These findings have not received Flathub approval. Follow the
[Flathub permission and linter review process](https://docs.flathub.org/docs/for-app-authors/linter)
before submission. Permission moderation and linter exceptions are separate
checks. Recheck with the linter used by Flathub at submission time because its
rules can differ from the installed version.
