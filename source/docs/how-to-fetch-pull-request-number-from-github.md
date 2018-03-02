---
layout: post
title: How to fetch the pull request number from Github repository?
category: Adapting Semaphore
---

Semaphore starts a [build after a pull request](/docs/building-pull-requests.html)
has been opened between the main and the forked repository.
Every time a new pull request is opened from a fork, Semaphore creates a new branch
whose name matches the pull request title and number.
Also, an [environment variable](/docs/available-environment-variables.html)
PULL_REQUEST_NUMBER is set.

Since Semaphore automatically builds branches that are created
within a repository, it doesn't support building pull requests that are opened
within the same repository. In this case, variable PULL_REQUEST_NUMBER is not set.

In order to fetch details from the pull request, we'd suggest using
the [Octokit gem](https://github.com/octokit/octokit.rb) and Github OAuth token
for this purpose. The following steps illustrate how to achieve this.

### Step 1: Adding Github OAuth token to your Semaphore project

[Create environment variable](/docs/exporting-environment-variables.html)
GITHUB_TOKEN in the project settings. Its value should be set to the generated token
on Github (Settings > Developer settings > Personal access tokens).
User token has to have the "repo" access.

ToDo: suggest "machine user"

### Step 2: Add script to Semaphore environment

ToDo: Describe script adding

```ruby
#!/bin/env ruby

require 'octokit'

c = Octokit::Client.new(:access_token => ENV['GITHUB_TOKEN'], :auto_paginate => true)
prs = c.pull_requests(ENV['SEMAPHORE_REPO_SLUG'], :state => "open")
current_pr = prs.select {|pr| pr[:head][:ref] == ENV['BRANCH_NAME']}

pr_value = current_pr.first[:number].to_s unless current_pr.nil?

puts pr_value
```

### Set 3: Set PULL_REQUEST_NUMBER

Environment variable can be set by adding these commands to the setup of the build

```bash
gem install octokit --no-ri --no-rdoc
export PULL_REQUEST_NUMBER=$(ruby pull_request_number.rb)
```
