# ServeIt

ServeIt is a tool for previewing content in a browser, automatically rebuilding it when things change.
It replaces tools like [guard](https://github.com/guard/guard) that do asynchronous rebuilds when files change.

ServeIt is strictly synchronous: it performs builds when requests come in, blocking the request until the build completes.
This means that the served content will never be stale or inconsistent, as it can be with asynchronous tools.

You can use ServeIt to preview a markdown file as you write it, to serve your blog as you write a post, to review a book as you edit it, etc.

## Usage

To serve the current directory at `http://localhost:8000` as a purely static site, including a simple directory listing, run:

```
$ serveit
```

That's useful, but most content isn't static.
I'm using the command below to preview this README as I write it.
Whenever I refresh the page, ServeIt runs `multimarkdown` to regenerate the HTML:

```
$ serveit 'multimarkdown README.md > README.html'
```

(You should try this yourself; it will make ServeIt's purpose and behavior much more clear than simply reading.)

When I go to `localhost:8000`, the `multimarkdown` command will be run, then I get a simple directory listing:

> <p><h3>Listing for /</h3></p>
> <a href="/..">..</a><br>
> <a href="/.git">.git</a><br>
> <a href="/.gitignore">.gitignore</a><br>
> <a href="/README.html">README.html</a><br>
> <a href="/README.md">README.md</a><br>
> <a href="/serveit">serveit</a><br>

If I click on `README.html`, I see this README file rendered as HTML.
When I load or reload any page, ServeIt will check for changes to any files in its current working directory.
If there was a change, *no matter what the change is*, then the command will be re-run, regenerating the HTML file.
The HTTP request only completes once the build is finished.

## Theory of Operation

ServeIt does exactly two things:

1. Serve static files.
2. Run a command when anything changes on disk, which may change the static files that it's serving.

ServeIt works well for building more complex content, like blogs or books.
However, it has no advanced concepts like dependency tracking; those are jobs for other tools.

You can use whatever build tool you like -- `serveit make`, `serveit rake`, `serveit grunt`, etc., and these tools can do arbitrary build tasks.
While writing this README, I'm sloppily letting ServeIt serve its own git repository directory, including detritus like the `.git` directory.
When working on my book, I have a more rigorous build system.
It uses `rake` rules to rebuild only chapters that have changed, and output goes into a `build` directory that doesn't contain any of the source files.

ServeIt doesn't know about any details of the build.
It will never be extended to know details of the build.
*ServeIt does not do builds. It serves files and runs commands when files change!*

## Is It Fast Enough?

ServeIt's file change tracking is naive: it simply crawls the current directory recursively, building a list of paths and their modification times.
Computers are fast now; this is not a performance problem.
My machine takes about 30 ms to check for changes in a directory of 10,000 files.
If your blog has 1,000,000 files in it, ServeIt will be slow.
Don't do that.

ServeIt adds a trivial amount of overhead to requests -- generally less than a millisecond.
If the build becomes annoyingly slow, that's a build tool problem that can be solved with incremental builds using `make`, `rake`, or whatever tool you like.
Or, even better, it can be solved by reducing the complexity of the build.
Reducing build complexity is usually more effective and reliable than increasing complexity by adding subtle incremental build rules.

## Options

You can change ServeIt's server root with `-s`.
This is useful when your build output goes into a specific directory.
To build and serve my book, I run this command in the root of its repo:

```
serveit -s build rake
```

This runs `rake` in the repo root, where there's a Rakefile.
When I reload, the Rakefile incrementally builds the book into the `build` directory.
ServeIt is serving the `build` directory because of the `-s` argument, so I see my rendered changes upon reload.

## Practicalities

ServeIt requires Ruby 1.9.3 or later, but has no other dependencies.
