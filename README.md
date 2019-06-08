# Useful documentation

After a lot of thinking, following the example of [@niqdev][niqdev], I've decided to create this project to collect all the useful information about what I like most: **programming**.

[niqdev]: https://github.com/niqdev/devops

### How to serve, build and deploy

To start the dev server:

```sh
$ mkdocs serve
INFO    -  Building documentation...
INFO    -  Cleaning site directory
[I 160402 15:50:43 server:271] Serving on http://127.0.0.1:8000
[I 160402 15:50:43 handlers:58] Start watching changes
[I 160402 15:50:43 handlers:60] Start detecting changes
````

In order to build the website:

```sh
mkdocs build
```

This will create a new directory, named `site`.


To deploy the website to GH pages:

```sh
mkdocs gh-deploy
```

#### Customization

[Material Theme](https://squidfunk.github.io/mkdocs-material)