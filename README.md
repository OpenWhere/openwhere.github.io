# OpenWhere.github.io
Public blog of OpenWhere, Inc.

## [Installing Jekyll](https://help.github.com/articles/using-jekyll-with-pages/#installing-jekyll)
+ Install Ruby (`brew install ruby`)
+ `gem install bundler`
+ `bundle install` (from the root directory of this project)

## [Running Jekyll Locally](https://help.github.com/articles/using-jekyll-with-pages/#running-jekyll)
```
cd ./blog
bundle exec jekyll serve --watch
```

## Writing Blog Posts
All posts are written in markdown and live in the `blog/_posts` folder.

Blog post markdown filenames **must** have a date (`YYYY-mm-dd`) prefix:
```
2014-10-19-post-title.markdown
```

Additionally, the posts must have a *front-matter* header that contains
metadata about the post, contained within three dashes:
```
---
layout: post
title:  "Sample Post"
date:   2014-10-09 13:44:31
categories: sample post
---

# Blog Post Title
Blog post content blah blah...
```

You can omit the front-matter from your post, but you still must include the
three dashes:
```
---
---

# Blog Post Title
Blog post content blah blah...
```

A sample post has been provided for you in the `_drafts` folder. Feel free to
copy it to the `_posts` folder, but make sure you change the title to match
today's date.
