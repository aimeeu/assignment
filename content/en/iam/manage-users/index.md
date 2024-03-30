---
title: Invite and manage users
linktitle: Users
description: >
  Use the Armory CD-as-a-Service Console to invite a user to your CD-as-a-Service organization.
toc_hide: true
hide_summary: true
exclude_search: true
---

## {{% heading "prereq" %}}

* If you don't want to assign the `Organization Admin` role to your new user, create a new role. See {{< linkWithTitle "iam/manage-rbac-roles.md" >}}.
* You need the user's name and email address. The email domain must match your organization's format. For example, users that work for Acme (which uses `username@acme.com`) must have `@acme.com` email addresses. Users are automatically added to your organization once they accept the invite and complete the sign up.

{{% alert title="Important" color="warning" %}}

* **A user can belong to only one CD-as-as-Service Organization (company account)**. If you try to invite a person that already belongs to another organization, you get a `409 Conflict: The user already exists` error.
* A user can have access to multiple tenants within a single organization.

{{% /alert %}}

## Invite a user

1. Access the CD-as-a-Service Console.
1. Navigate to **Access Management** > **Users**.
1. Click **Invite Users**
1. Enter the new user's full name in the **Name** field and the user's email address in the **Email** field.
1. Select at least one role from the **Roles** drop down list.
1. Click **Send Invitation**.
1. A modal window opens. Review the information and click **OK** to send the information or **Cancel** to return to the previous screen.

The new user receives an email with instructions for accessing the CD-as-a-Service Console.

### Two-Factor authentication

CD-as-a-Service requires two-factor authentication (2FA) when logging in. When setting up 2FA, a new user can use any 2FA authenticator that can scan a QR code. 

## Assign a role

1. Access the CD-as-a-Service Console.
1. Navigate to **Access Management** > **Users**.
1. Find the user you want to update. Click the **pencil icon** to open the **Edit User** screen.
1. In the **Edit User** screen, place your cursor in the **Roles** field and click.
1. Select a role from the drop-down list. Repeat if you want to assign the user more than one role. Selected roles appear below the **Roles** drop-down list.
1. Click the **Save Roles** button.

## Revoke a role

1. Access the CD-as-a-Service Console.
1. Navigate to **Access Management** > **Users**.
1. Find the user you want to update. Click the **pencil icon** to open the **Edit User** screen.
1. In the **Edit User** screen, you can see a user's roles listed below the **Roles** field.
1. Each assigned role has an **x** next to it. Click the **x** to revoke the role.

{{< figure src="user-role-delete.jpg" alt="User role with arrow pointing at 'x' to delete" height="80%" width="80%" >}}
