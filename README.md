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

Finally, build the website:
```bash
bundle exec jekyll build
```

This will create a `_site` directory which contains the website, as a set of static assets.

Alternatively, you can also serve the website locally:
```bash
bundle exec jekyll serve
```

You can then browse to [http://127.0.0.1:4000](http://127.0.0.1:4000) to visualise the website locally.
Although this should not be used in production, this is very handy when developing the website, as it is automatically rebuilt whenever a file changes.

## About
Private Data Donor is a research project whose goal is to gather statistics about Web search queries in a privacy-preserving way.
Collected data is then used to help monitoring and predicting outbreaks of infectious diseases such as flu.
It is developed by [UCL's CS department](http://www.cs.ucl.ac.uk/home/), in the frame of the [i-sense project](https://www.i-sense.org.uk/), the EPSRC IRC in Early Warning Sensing Systems for Infectious Diseases.

## License
This project is made available under the Creative Commons Attribution 4.0 International License.
