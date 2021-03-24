---
title: "Chef without Chef server"
date: 2015-09-14T12:00:00+02:00
categories: ["index", "chef", "ruby"]
---
There are couple alternatives available if you wish to use Chef in your infrastructure, but don't have budget to setup hosted Chef service. So far I've seen one based on Git and another based on AWS S3.
<!--more-->
## Git based solution

In git based solution, create repository into bitbucket.com and create directory tree:

* /chef/cookbooks/my_cookbook/definitions/default.rb
* /chef/cookbooks/my_cookbook/files
* /chef/cookbooks/my_cookbook/recipes/default.rb

In your _/chef/cookbooks/my_cookbook/recipes/default.rb_, put following text among other stuff:

```
directory '/root/bin' do
  owner 'root'
  group 'root'
  mode '0700'
  action :create
end

cookbook_file '/root/bin/run_chef' do
  source 'run_chef'
  owner 'root'
  group 'root'
  mode '0700'
  action :create
end

cron 'run chef hourly' do
  minute '13'
  user 'root'
  home '/root'
  command '/root/bin/run_chef'
  action :create
end
```

Create _/chef/cookbooks/my_cookbook/files/run_chef_:

```
#!/bin/bash
cd /root/my_repository/chef
git pull
if [ ! -f /etc/chef/client.rb ]; then
  mkdir -p /etc/chef
  cat > /etc/chef/client.rb &lt;&lt;EOF
log_level        :info
log_location     STDOUT
EOF
fi
chef-client -z -c /etc/chef/client.rb -o my_cookbook
```

Now you are ready to commit and push your git repository into bitbucket.

In client hosts, create ssh keys (without passphrase). Add your public ssh key as deployment key into your new repository. `git clone` your repository under /root. `cd my_repository` and `chef-client -z -o my_cookbook`

## S3 based solution

Our S3 based solution starts in same fashion with Git based solution. First we create git repository, create directory tree into it and configure cron to run /root/bin/run_chef on hourly basis. Things split into new course in _/chef/cookbooks/my_cookbook/files/run_chef_. With S3, file content should be:

```
#!/bin/bash
export PATH=/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
cd /root/
rm -rf cookbooks
aws s3 --no-sign-request cp s3://my_s3_bucket/my_cookbooks.tar.gz .
tar xzvf cookbooks.tar.gz
if [ ! -f /etc/chef/client.rb ]; then
  mkdir -p /etc/chef
  cat > /etc/chef/client.rb &lt;&lt;EOF
log_level        :info
log_location     STDOUT
EOF
fi
chef-client -z -c /etc/chef/client.rb -o my_cookbook
```

*NOTE:* awscli 1.2.9 doesn't support `--no-sign-request` command line option, so you need something more recent than that. If you install it with `pip install awscli` in Ubuntu, it will get installed under _/usr/local/bin_.

In your _/chef/cookbooks/my_cookbook_ directory, issue `berks install` and `berks package`. This should create _cookbooks-*.tar.gz_. Rename it into _cookbook.tar.gz_.

You also need S3 bucket my_s3_bucket with following kind of S3 bucket policy.

```
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AddCannedAcl",
      "Effect": "Allow",
      "Principal": {
          "AWS": "arn:aws:iam::MY_AWS_ACCOUNT_NUMBER:root"
      },
      "Action": [
          "s3:GetObjectAcl",
          "s3:GetObject"
      ],
      "Resource": "arn:aws:s3:::my_s3_bucket/*"
    }
  ]
}
```

Now you are ready to upload your _cookbook.tar.gz_ into my_s3_bucket. Remember to set _cookbook.tar.gz_ public, so that our EC2 instances can access it. Now you just have to copy _/chef/cookbooks/my_cookbook/files/run_chef_ to your instance (for example as part of userdata) and execute it once.

## Comparison

In Git based solution, prerequirements are git and chef-client. You have complete audit trail, because all content is pushed through the git repository, but you have to maintain deployment keys (or put one valid deployment keypair into chef configuration, if you want to do autoscaling).

In S3 based solution, prerequirements are awscli (that supposed --no-sign-request) and chef-client. You don't have to worry about deployment keys and you can use Berksfile to maintain all your 3rd party cookbooks, but you have to create new cookbook.tar.gz whenever you make changes to your chef configuration and there is no direct linking between your cookbook.tar.gz files and content in git repository.

Both scenarios work very well, if the goal is to ensure that all your nodes have necessary user accounts, properly configured sudoers and monitoring tools like nagios plugins or zabbix-agent.
