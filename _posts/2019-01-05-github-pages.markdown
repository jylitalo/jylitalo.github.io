---
layout: post
title: "Github Pages"
status: publish
date: 2019-01-05 12:00
---
I have maintained Jekyll based static blogs in AWS S3, Google's Cloud Storage and I felt that it might be interesting experiment in Christmas holidays to see what would be best way to do it with GitHub Pages. This seemed as an interesting idea, since GitHub Pages allows you to maintan your blog without locally installed Jekyll.

However if you want to put GitHub in charge of html creation, it has some downsides because your Jekyll version depends on what github-pages plugin supports. Also your theme and plugin selection is also partly limited, which can be a bonus as well, since it allows you to change theme without any changes into repository content. One last problem, that I was able to identify is that it is harder to preview your content (vs. local content creation and simply pushing the final site to GitHub).

With locally installed jekyll, you have to find a way to integrate `_site` subdirectory into master branch of your GitHub repository. My solution was to create local jekyll installation into `source` branch, define `_site` as submodule that points to `master` branch of repository. Final outcome can be seen at [jylitalo.github.io](http://github.com/jylitalo/jylitalo.github.io/) repository.
