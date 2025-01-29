# Guide for Data and Computation Team Members

This is a short write up facilitate the spin up of new team members for the Data and Computation Team and describe regular maintenance tasks.

## Onboarding

### Checklist for new members

- [ ] Ask to be added to the [Data and Computation Github team](https://github.com/orgs/leap-stc/teams/data-team/members)
- [ ] Ask to be added to the `@data-and-compute` Slack user group
- [ ] Subscribe to [](onboarding.slack)
- [ ] Consider enabling notifications for [](onboarding.github)
- [ ] Make a PR to the `_config.yaml` file [here](https://github.com/leap-stc/leap-stc.github.io/blob/fd69890ffc2f1871968e39b1c460370a0b3f98b3/book/_config.yml#L40-L51) in a PR. to add a picture and your personal data to the webpage.
- [ ] Get access to the [Grafana Dashboard](https://grafana.leap.2i2c.cloud)
- [ ] Request access to a service account to monitor Google Dataflow and Storage from the [Google Cloud Console](https://console.cloud.google.com/welcome?project=leap-pangeo) by raising an issue [here](https://github.com/leap-stc/data-and-compute-team/issues)
  - Instructions for admin:
    - Go to the Google Cloud Console > IAM > Grant Access
    - Add the following permissions:
      - `Dataflow Admin`
      - `Storage Admin`
      - `Logs Viewer`
      - `Monitoring Viewer`
      - `Logs Viewer`
      - `Compute Viewer`

(onboarding.slack)=

### Relevant Slack channels

- [`data-and-computation-team`](https://leap-nsf-stc.slack.com/archives/C065KPT1S4Q): Private channel for internal discussions, please contact the Manager for Data and Computing to be added.
- [`leap-pangeo`](https://leap-nsf-stc.slack.com/archives/C02KB0DDB6E): Community wide channel where LEAP-Pangeo users can ask questions. Members of the team should join the channel and regularly engage with issues raised.

(onboarding.github)=

### Relevant github repos

- [`leap-stc.github.io`](https://github.com/leap-stc/leap-stc.github.io): The source for the [LEAP-Pangeo technical documentation](https://leap-stc.github.io/intro.html). This also contains [LEAP-Pangeo Discussion Forum](https://github.com/leap-stc/leap-stc.github.io/discussions)
- [`data-management`](https://github.com/leap-stc/data-management): Contains all the [pangeo-forge](https://pangeo-forge.readthedocs.io/en/latest/) based code/automation to ingest new datasets. Point users to the [issue tracker](https://github.com/leap-stc/data-management/issues/new/choose) to request new datasets.

## Regular Maintenance

This section documents common tasks performed as part of the Data and Computation Team's duties.

### Monitor User Directory Useage

The [user directories](reference.infrastructure.hub.user_dir) of all members are on a shared volume of fixed size. We pay for the total size of the volume no matter if it is used or not, so we should strive to keep usage to a minimum. Our current policy is that \<50GB per user is acceptable.
Unfortunately we do not have a technical way to implement per user quotas, so we need to regularly check two things:

- How much space is left on the [entire shared volume](https://grafana.leap.2i2c.cloud/d/hub-dashboard/jupyterhub-dashboard?orgId=1&viewPanel=23)? If the free space falls *below 20%* we need to discuss further actions.
- [Which users are taking up more than the allowed 50GB](https://grafana.leap.2i2c.cloud/d/bd232539-52d0-4435-8a62-fe637dc822be/home-directory-usage-dashboard?orgId=1). For users who take up more than 50GB the suggested action is to reach out via slack or email.

### Regularly Updating the Software Environment

We aim to provide users with [up-to-date default software environments](reference.infrastructure.hub.software_env). This currently requires to change the [pangeo-docker-images](https://github.com/pangeo-data/pangeo-docker-images) tag manually.

- Make sure you are subscribed to release notifications on [pangeo-docker-images](https://github.com/pangeo-data/pangeo-docker-images) to recieve Github notification about new releases
- To bump the version, submit a PR to [this config file](https://github.com/2i2c-org/infrastructure/blob/master/config/clusters/leap/common.values.yaml) in the [2i2c infrastructure repo](https://github.com/2i2c-org/infrastructure). In that PR you need to change the image tag for *all image choices*, see an example [here](https://github.com/2i2c-org/infrastructure/pull/3688).
- To send emails, a token file is setup as the OAUTH_GMAIL_CREDENTIALS Github Secret in the member management repo; every so often (around biweekly) the action will require re-authentication, generating a new token that should replace the existing secret. This makes use of OAUTH_GMAIL_CLIENT_SECRET, which never needs to change. To update the OAUTH_GMAIL_CREDENTIALS secret, run the `generate_emails_token(github_action_mode=False)` function from utils.py locally, which will direct you to a confirmation screen. Log in to the leap.pangeo@gmail.com account and authorize access. You will require a copy of the CLIENT_SECRETS file on your personal machine which can be retreived from Google Cloud Console from the above pangeo support email.

## Offboarding members

- [] Delete personal `dct-team-<first_name>` service account in IAM (needs admin priviliges).

## Admin Tasks

This part of the guide is reserved for team members with admin access to the `'leap-stc'` github organization!

(guide.team.admin.renew_member_token)=

### Renewing Personal fine grained access token for LEAP member management

In order to automate member sign up by adding github users from a private Google Sheet to the appropriate github teams (via [this gh action](https://github.com/leap-stc/member_management/blob/main/.github/workflows/read_sheet.yaml)) the github action needs the appropriate priviliges to add/remove members from teams. We are currently handling this by providing a personal access token as the `"ORG_TOKEN"` secret. The person creating the token will usually be the Manager for Data and Computation.

:::\{note}
Ideally we want to remove the dependency on a single user account here, but for now this is the only way I have found this to work properly. Maybe there is a way to establish a 'dummy' user?
:::

#### Steps

- Make sure you have access to set secrets on the private [member_management repo](https://github.com/leap-stc/member_management)
- Go to the personal account "Settings>Developer Settings" Tab. From there naviate to "Personal Access Token>Fine-Grained tokens"
- If present click on "LEAP member management token", othewise create a new token with that name (the actual name is optional here, but make sure to name it in a memorable way), and authenticate.
- Generate or regenerate the token
  - The required permissions are "Read and Write access to members" and "Read Access to actions and metadata"
  - Set the expiration to a full year (the current limit set on the org level)
- Make sure to copy the token (leave the page open until the next step is completed, since you will have to recreate the token once the page is closed!)
- Go to the [member_management repo](https://github.com/leap-stc/member_management) and navigate to "Settings > Secrets and Variables > Actions" and open the "ORG_TOKEN" to edit
- Paste the above token from the clipboard and save.
- Run the [Member Add Action](https://github.com/leap-stc/member_management/actions/workflows/read_sheet.yaml) and confirm that it is successful
- Close the token page and you are done!

### Handover Checklist for Admins

The following is a list of tasks that should be done by any new hire in the Data and Computation Manager position to ensure smooth operations.

- [](guide.team.admin.renew_member_token)

## Non-Technical Admin Tasks

This section describes admin tasks that are necessary for the maintenance of LEAP-Pangeo components (including collaborative efforts lead M²LInES) which require appropriate permissions, but no coding (everything can be achieved on one of several websites).

(guide.team.admin.member_signup_troubleshooting)=

### Member Sign Up Troubleshooting

Ideally members should be signed on automatically and well ahead of any event (see [](reference.member_sign_up) for an overview of our member sign up mechanics). But despite the best efforts situations arise where either staff or the organizers/instructors of an event need to quickly sign on new members, and troubleshoot if certain users do not have access to the JupyterHub. Follow the steps in this Flowchart to quickly resolve any issues in such cases.

![](../images/member_troubleshooting.png)

#### A+G: Check if a user is in the Member Spreadsheet

Request read access or inquire with LEAP staff to confirm that the user(s) who are having trouble, are listed in the Member Data Spreadsheet and make sure the the sheet **includes their github username**. Always make sure that the github name exists (search in github), and does not contain extra characters like `"@"`.

#### B: Add Users to the Member Spreadsheet

If this is not time sensitive, make sure that the user has completed the membership application (more details [here](reference.membership.tiers)) and inquire with LEAP staff about the status of membership (adding users might take a while, thus always sign up users well in advance). In a time sensitive situation contact LEAP staff to expedite the addition of new users added to the Member Spreadsheet.

#### C+D+F: Checking Github Team Membership and invite status

Repeat the following steps for all relevant Github Teams (you can find links to the team page [here](reference.membership.tiers)):

- C: Navigate to the search bar at the top that says "Find a member..." and enter the **github username** of the user. If the name shows up the user is part of this github team
- D: Right next to the search bar is a button that says "... pending members". Click on that button and scroll down the list. If the username is in that list the member has received an invite, but not yet accepted it.
- F: **Accepting the invite has to be done by the user!**. Point them to our [FAQ's](faq.where_is_my_invite) for instructions on how to accept the invite.

#### E: Manually rerunning github sign up action

:::\{attention}
You need to have maintainer access to the private [leap-stc/member_management repo](https://github.com/leap-stc/member_management) in order to follow these steps. If you cannot repeat these steps please ask one of the github organization admins to be added to an appropriate team (e.g. [bootcamp-instructor](https://github.com/orgs/leap-stc/teams/bootcamp-instructors)) that has access.
:::

- Navigate to [leap-stc/member_management](https://github.com/leap-stc/member_management)
- In the top rider click on "Actions"
- On the left click on "Parse Member Sheet and Add New Members"
- In the upper right corner, click on "Run Workflow", and again on "Run Workflow" in the pop up window.
- After a short while you should see a yellow circle in the main window, indicating that the github action is in progress. Wait until the action is finished (usually 3-5 minutes).
- If the circle turns green, you are done. If the circle turns red, try to run it one more time to exclude any random issues. If the error persists, reach out to a member of the [](support.data_compute_team)

#### H: Clearing Browser Cache etc

If users are part of a github team, and still have trouble signing on, instruct them to clear their browser cache, or try a different browser. This step is unlikely to be necessary

#### If none of this works

:::\{note}
This is a very unlikely scenario and has very rarely happend!
:::
Follow the flowchart and reach out to either the [](support.data_compute_team) or [2i2c support](https://docs.2i2c.org/support/).

### M²LInES OSN pod administration

All administrative tasks pertaining to the M²LInES OSN bucket are handled via the [Coldfront Portal](https://coldfront.osn.mghpcc.org/). Please log in with one of your affiliated organizations and make sure to *use the same one each time in case you have multiple affiliations*.

The important units of division on the OSN pod are **projects** and **buckets**. Each project can have multiple buckets, and you can give others access as guests and admins on a per project basis. Buckets are how the actual storage space is organized, and each bucket will have *access credentials* and a *storage quota*, both of which might need actions from an admin from time to time.

(guide.team.admin.osnpod.check)=

#### Check Bucket attributes

To check individual buckets' attributes log into the [Coldfront Portal](https://coldfront.osn.mghpcc.org/), click on the relevant project, navigate to the "Allocations" section, find the bucket name in the "Information" column and click on the folder symbol in the "Actions" column.

Scroll to the "Allocation Attributes" section. You can see all relevant values here.

**OSN Anonymous Access**: If False, this data is public, no credentials are needed to read data (writing still requires credentials).
**OSN Bucket Quota (TB)**: Shows the currently allocated size. This is the max size, not what is actuall used!
**OSN RO/RW Bucket Access/Secret Key**: Credentials for read-only (RO) and read-write (RW) access to the bucket. See [](reference.infrastructrue.osn_pod.credentials) for more details.

#### Share bucket credentials

:::\{attention}
Some buckets are not meant to be accessible for write by users! Please always refer to [](reference.infrastructrue.osn_pod.organization) and only give access to project specific buckets and the `'leap-pangeo-inbox'` bucket to non-admins.
:::

- Navigate to the specific bucket you want to share credentials to (see [above](guide.team.admin.osnpod.check) for detailed steps)
- Copy the relevant Access and Secret keys (either RO or RW depending on the desired use) and share them with the relevant users e.g. by pasting them into a password manager and sharing an authenticated link.

#### Increasing Storage Quota

If any of the buckets needs more storage space, follow these steps:

- Log into [Coldfront](https://coldfront.osn.mghpcc.org/)
- Navigate to the project that contains the bucket (we currently separate projects for M²LInES, LEAP, and the LEAP-Pangeo Data ingestion)
- Scroll to "Allocations" and find your bucket in the "Information" column. Click on the folder icon in the corresponding "Actions" column.
- In the top right click on "Request Change" and scroll down to "Allocation Attributes". Enter the desired new size in TB in the "Request New Value" column and the "OSN Bucket Quota (TB)" row and enter a short justification (required).
- Click the "Submit" button.
- You should see a green box with "Allocation change request successfully submitted. " at the top of the next page.
- Wait for email confirmation of the change.

#### Provision a new Bucket

- Log into [Coldfront](https://coldfront.osn.mghpcc.org/)
- Navigate to an existing Project (or create a new one; see below), scroll to "Allocations" and click on "Request Resource Allocation"
- In the following dialog chose "OSN Bucket (Storage)", write a short justification which **includes whether you want the bucket to have anonymous (public) access!**, and choose a size in TB.
- Click "Submit"
- Wait for email confirmation.

#### Create a new Project

:::\{note}
You need PI status on the pod to create new projects. Reach out to the M²LInES admin to discuss this if you do not have access yet.
:::

- Log into [Coldfront](https://coldfront.osn.mghpcc.org/)
- Click on the "Projects" link on the homepage
- Click "Add a Project"
- Choose a title, write a short description of the project, and optionally choose a field of science.
- Then click "Save"
- Wait for email confirmation.

#### OSN Support

For any questions/functionality not mentioned above, please refer to the [OSN documentation](https://coldfront.osn.mghpcc.org/static/osndocs/index.html) or reach out to the OSN support (`'help@osn.mghpcc.org'`)
