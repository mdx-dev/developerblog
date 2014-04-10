# Developer Blog

## Theme Readme
For the theme's readme, view `/_layouts/README.md`.

## Adding yourself as a author
To add yourself as an author edit the `/_config.yml`. Make sure to add all the necessary fields under `authors` and add your `key` to the `author_ids`.

## Generating your Gravatar
I added a rake task:
~~~
-> % rake gravatar
Email your email address: blainesch@gmail.com
http://www.gravatar.com/avatar/ffebdf905fae3278a260ed03c9d165c7
~~~

## Writing a post
[View the jekyll documentation](http://jekyllrb.com/docs/posts/)

## Workflow
Make a pull request to master with new authors, posts, or other edits.

## Making changes to CSS (via LESS)
Make your changes to the necessary LESS file in assets/less
in the terminal, run grunt
then run jekyll build
then run jekyll serve to view your changes on localhost.
commit your changes to github when finished.