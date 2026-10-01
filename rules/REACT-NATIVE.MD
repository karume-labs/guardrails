# React Native / Expo Projects

- **No editing native Android and iOS folders directly.** Direct modifications to the `android/` and `ios/` directories are strictly banned.
- When native configuration or dependency linking is needed, use Expo plugins via `app.json` (or `app.config.ts`), or create custom Expo config plugins.
- Rely on Continuous Native Generation (CNG) to manage the native directories.
