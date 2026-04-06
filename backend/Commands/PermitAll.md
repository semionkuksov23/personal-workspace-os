# Command: PermitAll

## Purpose
Enable full permission mode for the current session so routine steps do not require per-action approval.

## Safety Note
This command reduces interaction friction but does not authorize destructive or irreversible actions without explicit user intent.

## Steps
1. Acknowledge permit-all mode activation.
2. Continue task flow without requesting routine confirmations.
3. Still require explicit confirmation for:
   - destructive deletes outside defined retention workflows
   - bulk irreversible operations
   - actions with legal/compliance consequences if ambiguous
4. Remind user permit-all can be revoked any time.

## Output
- `PermitAll mode active for this session.`
