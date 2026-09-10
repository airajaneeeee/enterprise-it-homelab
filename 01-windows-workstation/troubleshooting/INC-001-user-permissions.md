# INC-001 — User Unable to Perform Administrative Operation

## Scenario

A Finance department employee attempted to perform an operation requiring administrator privileges.

## User Impact

The user could perform normal workstation tasks but could not make a protected system change.

## Investigation

1. Confirmed the logged-in account using:

   ```cmd
   whoami
   ```

2. Reviewed local administrator membership using:

   ```cmd
   net localgroup administrators
   ```

3. Confirmed that `finance.user` was configured as a standard user.

## Root Cause

The operation required elevated privileges, while the employee account intentionally had standard-user permissions.

## Resolution

The required administrative task was performed using authorized administrator credentials rather than permanently granting the employee administrator access.

## Verification

The administrative operation completed successfully while `finance.user` remained a standard user.

## Security Principle

This configuration follows the principle of least privilege.

## Lessons Learned

- Access-denied errors can indicate intentional permission restrictions.
- Administrative access should not be granted permanently simply to complete a single privileged operation.
- User identity and group membership should be verified before changing permissions.