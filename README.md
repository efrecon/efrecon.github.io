# efrecon.github.io

This is the source for my blog, the theme was originally forked from
[so-simple](https://github.com/mmistakes/so-simple-theme). Thanks!

## Running Locally

To run locally, place yourself in the main directory of the repository and run
the following:

```console
docker run -t --rm -v "$PWD":/usr/src/app -p "4000:4000" starefossen/github-pages
```

Then direct your browser to [http://localhost:4000](http://localhost:4000).  The
[image](https://github.com/Starefossen/docker-github-pages) supports reload so
any change that you make will propagate to the site.