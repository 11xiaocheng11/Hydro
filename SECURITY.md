## Summary

Several high-impact administrative actions appear to require only `PRIV_EDIT_SYSTEM` but do not enforce `requireSudo` re-authentication, while other sensitive admin routes in the same file do require sudo confirmation.

This leads to inconsistent protection for dangerous actions such as restart, script execution, and bulk user import.

## Impact

If an admin session is stolen, left open, or used in a shared environment, an attacker may be able to:
- restart the service
- execute system scripts
- bulk import accounts and modify groups

without triggering the extra sudo confirmation that is already used elsewhere for sensitive operations.

## Code References

### Sensitive actions without `requireSudo`

- `packages/hydrooj/src/handler/manage.ts:80-89`
  - `postRestart()` executes `pm2 reload`
- `packages/hydrooj/src/handler/manage.ts:92-131`
  - `SystemScriptHandler.post()` executes `global.Hydro.script[id].run(...)`
- `packages/hydrooj/src/handler/manage.ts:235-307`
  - `SystemUserImportHandler.post()` performs bulk user creation and group updates

### Sensitive actions with `requireSudo` in the same file

- `packages/hydrooj/src/handler/manage.ts:134-170`
  - system settings
- `packages/hydrooj/src/handler/manage.ts:173-231`
  - config editing
- `packages/hydrooj/src/handler/manage.ts:314-349`
  - user privilege management

### Sudo implementation

- `packages/hydrooj/src/service/server.ts:47-73`
  - `requireSudo` decorator implementation

## Why this is a security issue

The project already recognizes the need for step-up authentication through `requireSudo`, but the protection is applied inconsistently.

Restarting the service, executing system scripts, and bulk importing users are all high-impact actions and should be protected at least as strictly as settings or privilege pages.

## Suggested Fix

- Apply `@requireSudo` consistently to all high-impact administrative actions.
- Review all `/manage/*` handlers for step-up authentication consistency.
- Consider documenting a security policy for which actions must require sudo confirmation.

## Additional Notes

This report is based on code review of:
- `packages/hydrooj/src/handler/manage.ts`
- `packages/hydrooj/src/service/server.ts`
