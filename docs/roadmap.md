# Roadmap

This roadmap is drawn directly from the honestly-labeled gaps in
[Capabilities](capabilities.md) — it isn't aspirational marketing copy. No
date commitments are made here; items are ordered by what's closest to
done, not by promised delivery.

## Near-term

- **Interactive UI verification for the packaged desktop build.** Every
  non-interactive aspect of the packaged Windows app (startup, backend
  health, real task execution, crash recovery, restart) is verified today
  through the app's own live API. What remains is verifying interactive
  mouse/keyboard behavior against that exact packaged build.
- **Cloud model (Anthropic) re-verification against a live API key.** The
  integration is implemented and covered by tests with a mocked provider;
  the next pass re-confirms it against the real Anthropic API end-to-end.
- **Durable, server-backed chat history.** Chat history is currently
  client-side only. A durable store is designed and scoped, not yet built.

## Mid-term

- **Computer Use live re-verification.** The real desktop-control backend
  (mouse, keyboard, screen capture, window management) builds correctly
  and is unit-tested against real automation libraries; the next pass
  re-confirms live, interactive execution against real hardware.
- **Remote/HTTP MCP servers.** Today's MCP client supports local
  (stdio) servers only.
- **Docker execution isolation, live-verified.** The container isolation
  backend is implemented; the next pass verifies it against a live Docker
  daemon.

## Longer-term

- **macOS and Linux packaging.** Today's packaged desktop build is
  Windows-only.
- **Code-signing for the installer and executable**, to remove the
  unsigned-binary warning users currently see.
- **Multi-user / team features** (shared mission history, shared approval
  workflows) — planned as part of a future commercial offering; see
  [Commercial / Partnership](../README.md#commercial--partnership) to
  inquire.

## Guiding principle

A capability moves from "planned" to "partial" to "verified" in
[capabilities.md](capabilities.md) only when it has actually been
exercised with independent evidence — not when the code for it merely
exists. This roadmap will be updated as that evidence is produced, not
ahead of it.
