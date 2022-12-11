---
title: "Linters"
date: 2022-12-11T12:00:00+02:00
categories: ["index", "software"]
---
Linter is any tool that detects and flags errors in programming languages, including stylistic errors. - [Wikipedia](https://en.wikipedia.org/wiki/Lint_(software))
<!--more-->
My favorite linters for various tasks:
* Ansible: [ansible-lint](https://github.com/willthames/ansible-lint)
* Golang:
  * [golangci-lint](https://github.com/golangci/golangci-lint) - catches dead code among other things
  * [gosec](https://github.com/securego/gosec) - really checks return values from functions
* JavaScript: [prettier-standard](https://www.npmjs.com/package/prettier-standard)
* Puppet: [puppet-lint](http://puppet-lint.com/)
* Python: [pylint](https://www.pylint.org/)
* Ruby: [rubocop](https://rubocop.readthedocs.io/en/latest/)
* Terraform: `terraform fmt`

## Changelog

* 2022-Dec-11: Added gosec
* 2021-Feb-21: Added golangci-lint
