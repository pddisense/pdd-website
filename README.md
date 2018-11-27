# pdd-website

This repository contains the code for the website presenting the PDD project and the technical documentation.
The website is created using Jekyll (a static website generator).

## Build
To build the website, you will need [Ruby ≥ 2.5.1](https://www.ruby-lang.org) and [Bundler ≥ 1.16.1](https://bundler.io).

First clone the repository:
```bash
git clone git@github.com:pddisense/pdd-website.git
cd pdd-website
```

Then install Jekyll and its dependencies (this has to be done only once, or when the dependencies change):
```bash
bundle install
```

Finally, build and serve the website locally.
```bash
bundle install #
bundle exec jekyll serve
```

You can then browse to [http://127.0.0.1:4000](http://127.0.0.1:4000) to visualise the website locally.
Although this should not be used in production, this is very handy when developing the website, as it is automatically rebuilt whenever a file changes.

## Release
There is a release script, which is a small helper to create a packaged website.

```bash
./bin/release
```

This will create a `dist/website.tar.gz` package containing only a set of static assets.
This will then need to be uploaded to the target Web server.
Alternatively, you can use the `-publish` flag on the release script to automatically deploy it to the official PDD server.
It assumes that there is a "pdd" server configured to be accessible via SSH, and that the website is deployed in the "~/website" directory.

## About
Private Data Donor is a research project whose goal is to gather statistics about Web search queries in a privacy-preserving way.
Collected data is then used to help monitoring and predicting outbreaks of infectious diseases such as flu.
It is developed by [UCL's CS department](http://www.cs.ucl.ac.uk/home/), in the frame of the [i-sense project](https://www.i-sense.org.uk/), the EPSRC IRC in Early Warning Sensing Systems for Infectious Diseases.

## License
This project is made available under the Creative Commons Attribution 4.0 International License.
