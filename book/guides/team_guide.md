# Guide for Data and Computation Team Members

This is a short write up facilitate the spin up of new team members for the Data and Computation Team and describe regular maintenance tasks.

## Onboarding

### Checklist for new members

- \[ \] Ask to be added to the [Data and Computation Github team](https://github.com/orgs/leap-stc/teams/data-team/members)
- \[ \] Ask to be added to the `@data-and-compute` Slack user group
- \[ \] Subscribe to [](onboarding.slack)
- \[ \] Consider enabling notifications for [](onboarding.github)
- \[ \] Make a PR to the `_config.yaml` file [here](https://github.com/leap-stc/leap-stc.github.io/blob/fd69890ffc2f1871968e39b1c460370a0b3f98b3/book/_config.yml#L40-L51) in a PR. to add a picture and your personal data to the webpage.
- \[ \] Get access to the [Grafana Dashboard](https://grafana.leap.2i2c.cloud)
- \[ \] Request access to a service account to monitor Google Dataflow and Storage from the [Google Cloud Console](https://console.cloud.google.com/welcome?project=leap-pangeo) by raising an issue [here](https://github.com/leap-stc/data-and-compute-team/issues)
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

- \[\] Delete personal `dct-team-<first_name>` service account in IAM (needs admin priviliges).

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
