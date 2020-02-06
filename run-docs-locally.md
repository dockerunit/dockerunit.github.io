## Requirements
- Docker 17+

## Running the docs
1. Clone this repo: `git clone git@github.com:dockerunit/dockerunit.github.io.git`
2. Enter the cloned repo: `cd dockerunit.github.io`
3. Run the Jekyll docker image: `docker run --rm -p 4000:4000 -it -v $PWD:/srv/jekyll jekyll/jekyll:3.8 sh -c "bundle update && bundle exec jekyll serve --host 0.0.0.0"`
4. Open your browser on `http://localhost:4000`



