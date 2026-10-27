# active-directory-onboarding-offboarding-lab
homelab, windows server 2025, it support.


**Active Directory Onboarding and Offboarding Lab**

A hands-on home lab where I built a small Windows Server domain (lab.local) and worked through common admin and help desk tasks: setting up accounts and groups, sharing a folder by team, enforcing password and lockout rules, onboarding and offboarding employees, and handling a locked out user.

All tickets below are simulated requests I wrote for myself to practice.

**Environment**
Hypervisor: VirtualBox
Server: Windows Server 2025, domain controller DC01
Client: Windows 11 VM joined to lab.local
Skills Practiced

Active Directory Users and Computers, organizational units, security groups, Group Policy, password and account lockout policy, shared folder permissions, account unlocks and password resets, documentation

**Ticket Log**
**Ticket #1: Set up domain structure and initial staff accounts**

Request: Build a basic structure for a new domain so accounts and access can be managed by team instead of one at a time.

What I did:

Created organizational units in lab.local: Staff for active employees, Disabled (Offboard) for departed employees, and Groups for security groups.
Created two security groups, Staff-Sales and Staff-Support.
Created five user accounts in the Staff OU: Robert Finlay, Jen Smith, Amber Jones, David Full, and Edgar Pow.
Added users to groups: Amber Jones, Jen Smith, Robert Finlay -> Staff-Support
                       Aaron May, David Full, Edgar Pow -> Staff-Sales

Result: The domain has a clear structure ready for group-based permissions, onboarding, and offboarding.

<img width="746" height="515" alt="image" src="https://github.com/user-attachments/assets/38ce565e-f332-4bff-89a3-0be5baecb9a8" />

<img width="608" height="311" alt="image" src="https://github.com/user-attachments/assets/59414bff-e7f7-49a6-b9f9-72e114b638d4" />

Domain structure and groups Staff OU users

**Ticket #2: Shared folder for Sales staff**

Request: Sales staff need one shared location for sales info, limited to the sales team.

What I did

Created the Salesinfo folder and shared it from the Sharing tab.
Gave the Staff-Sales security group Read/Write access instead of adding people individually, so future sales hires only need to be added to the group.
Left the Administrators group as owner.

Result: Access to Salesinfo is controlled through Staff-Sales membership. [Add: tested with a amay user and jsmith user outside the group, jsmith was unable to access documents within the shared folder.

<img width="580" height="448" alt="image" src="https://github.com/user-attachments/assets/bfee57ea-e58a-4bbf-bf99-910a5ff0cf7f" />
 
Share Salesinfo

**Ticket #3: Password and account lockout rules for Staff**

Request: Staff accounts need a consistent password and lockout standard.

What I did

First attempt: created a GPO called Staff Onboard Policy, set the password and lockout rules in it, and linked it to the Staff OU.
Learned that password and lockout policy for domain accounts has to be set at the domain level (or with a fine-grained password policy), not in a GPO linked to an OU. I moved the settings into the Default Domain Policy.
Password policy: minimum length 10 characters, complexity requirements enabled, maximum age 42 days. Password history is not enforced.
Lockout policy: lock the account after 4 invalid logon attempts, locked for 10 minutes, counter resets after 10 minutes. 
Administrator account lockout is set to Enabled to ensure attacker lockout in case of incident. Ideally root login access would be disabled alltogether to comply with Security+ safety standards.

Result: Password and lockout rules apply to domain accounts. See Ticket #6 for the lockout test.

<img width="751" height="508" alt="image" src="https://github.com/user-attachments/assets/aef608b1-eb13-420b-bf30-c0e53924c983" />

<img width="977" height="546" alt="image" src="https://github.com/user-attachments/assets/09efd634-41ad-43f2-84ad-317862d7afaa" />

<img width="968" height="546" alt="image" src="https://github.com/user-attachments/assets/ed14b618-e210-4e59-b588-55871246826b" />

Initial GPO link Domain password policy Domain lockout policy

**Ticket #4: Onboard new employee, Aaron May**

Request: New hire needs a domain account.

What I did

Created the account amay@lab.local in Active Directory Users and Computers.
Set "User must change password at next logon" so he sets his own password on first sign-in.
Left the account set to never expire.
Placed in the Staff OU and added to Staff-Sales.

Result: Account ready for first login.

<img width="412" height="525" alt="image" src="https://github.com/user-attachments/assets/5242e35a-7374-406a-8d02-3c7a0f15f2f7" />

New user account settings

**Ticket #5: Offboard departing employee, David Full**

Request: Employee has left the company and access needs to be removed.

What I did

Disabled the account instead of deleting it, so records and ownership are preserved.
Moved it from the Staff OU to the Disabled (Offboard) OU.
Removed group memberships and reset password to prevent the previous user from logging on

Result: David Full's account is disabled and sitting in the offboard OU.

<img width="736" height="521" alt="image" src="https://github.com/user-attachments/assets/6b96642b-1280-439c-8ee3-2b73f8a18b21" />

Offboard OU

**Ticket #6: User locked out, account unlock and password reset**

Request: Jen Smith cannot sign in and sees a lockout message.

What I did

Triggered the lockout by entering incorrect passwords at the sign-in screen, and got the "account is currently locked out" message. Confirmed that it only took 4 attempts.
Confirmed in Active Directory Users and Computers that the account showed as locked out on the domain controller.
Reset the password and checked "User must change password at next logon" so the user chooses their own.
Checked "Unlock the user's account" during the reset.
Signed in from the client and got the forced password change prompt.
Tested a short password and confirmed it was rejected, then set a valid one.

Result: Jen Smith can sign in again with a new password. The lockout policy from Ticket #3 works as configured.

<img width="572" height="444" alt="image" src="https://github.com/user-attachments/assets/81146243-38e5-4954-af90-9f93e1c2fb45" />

<img width="401" height="536" alt="image" src="https://github.com/user-attachments/assets/10183c64-9dd2-4221-9b58-2fab154507e8" />

<img width="392" height="146" alt="image" src="https://github.com/user-attachments/assets/7ec6635e-29bd-448f-b2ef-915cfc46d004" />

Lockout message Account locked in AD Forced password change

Issues I Ran Into
Problem: I set the password and lockout rules in a GPO linked to the Staff OU. The lockout then didn't trigger and I didn't know why. Fix: I learned that account policies for domain users have to be set at the domain level, so I rebuilt them in the Default Domain Policy and retested.
What I Learned

Group Policy applies differently depending on what you are configuring, and account policies are one of the exceptions to "link it to the OU." Giving access to groups instead of individual users makes onboarding and offboarding a lot cleaner. And a locked out user is a different ticket than a forgotten password, so I now check which one it is before touching anything.

