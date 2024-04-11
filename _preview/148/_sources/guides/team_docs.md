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

(onboarding.slack)=
### Relevant Slack channels
- [`data-and-computation-team`](https://leap-nsf-stc.slack.com/archives/C065KPT1S4Q): Private channel for internal discussions, please contact the Manager for Data and Computing to be added.
- [`leap-pangeo`](https://leap-nsf-stc.slack.com/archives/C02KB0DDB6E): Community wide channel where LEAP-Pangeo users can ask questions. Members of the team should join the channel and regularly engage with issues raised.

(onboarding.github)=
### Relevant github repos
- [`leap-stc.github.io`](https://github.com/leap-stc/leap-stc.github.io): The source for the [LEAP-Pangeo technical documentation](https://leap-stc.github.io/intro.html). This also contains [LEAP-Pangeo Discussion Forum](https://github.com/leap-stc/leap-stc.github.io/discussions)
- [`data-management`](https://github.com/leap-stc/data-management): Contains all the [pangeo-forge](https://pangeo-forge.readthedocs.io/en/latest/) based code/automation to ingest new datasets. Point users to the [issue tracker](https://github.com/leap-stc/data-management/issues/new/choose) to request new datasets.

## Regular Maintenance

### Monitor User Directory Useage

The [user directories](hub.guide.data.user_dir) of all members are on a shared volume of fixed size. We pay for the total size of the volume no matter if it is used or not, so we should strive to keep usage to a minimum. Our current policy is that <50GB per user is acceptable. 
Unfortunately we do not have a technical way to implement per user quotas, so we need to regularly check two things:
- How much space is left on the [entire shared volume](https://grafana.leap.2i2c.cloud/d/hub-dashboard/jupyterhub-dashboard?orgId=1&viewPanel=23)? If the free space falls *below 20%* we need to discuss further actions.
- [Which users are taking up more than the allowed 50GB](https://grafana.leap.2i2c.cloud/d/bd232539-52d0-4435-8a62-fe637dc822be/home-directory-usage-dashboard?orgId=1). For users who take up more than 50GB the suggested action is to reach out via slack or email.
