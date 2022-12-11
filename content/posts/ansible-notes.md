---
title: "Notes about Ansible"
date: 2017-12-06T12:00:00+02:00
categories: ["index", "python"]
---
My favorite Ansible tools and personal contributions towards Ansible.
<!--more-->

## Profiling Ansible

Create _ansible.cfg_ file into same directory with playbook with following content:
```
[default]
callback_whitelist = profile_tasks
```

Source: [ansible-profile](https://github.com/jlafon/ansible-profile)

## Related tools
- [ansible-lint](https://github.com/willthames/ansible-lint)

## Related tools

## Personal opinions, contributions, etc.
My personal background with configuration management tools starts from Cfengine in '90s that we used to customize Linux installations. Followed by Puppet to setup multi-node installations in '00s. In '16, I came across with Chef, but my current choice for configuration tasks is Ansible, even if it is not the fastest tool on a market.

### Contributions to other Ansible projects
- [ansible-sensu](https://github.com/sensu/sensu-ansible/pulls?utf8=%E2%9C%93&q=is%3Apr+jylitalo)

### Personal Ansible Roles projects
- [EmailRelay](https://github.com/jylitalo/EmailRelay)
- [GceInstance](https://github.com/jylitalo/GceInstance)
- [JekyllSetup](https://github.com/jylitalo/JekyllSetup)
- [SyncToGoogleStorage](https://github.com/jylitalo/SyncToGoogleStorage)
- [TwitterBot](https://github.com/jylitalo/TwitterBot)

### Personal Ansible Playbook projects
- [configure Macbook](https://github.com/jylitalo/configs)
- [blog.ylitalot.com site](https://github.com/jylitalo/blog.ylitalot.com-site)
- [ylitalot.com site](https://github.com/jylitalo/ylitalot.com-site)
- [ylitalot.net site](https://github.com/jylitalo/ylitalot.net-site)

### Pull requests to other Ansible projects
- [node-elb-log-parser](https://github.com/toshihirock/node-elb-log-parser/pull/2)
