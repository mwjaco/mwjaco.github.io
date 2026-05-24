# mwja.co

Static site built with [Minimal Mistakes Jekyll theme](https://mmistakes.github.io/minimal-mistakes/)

## Local Development Setup (macOS)

If `bundle install` fails while building the `eventmachine` native extension, the likely cause is that the Command Line Tools SDK (particularly CLT 26.x) does not expose C++ standard library headers to clang++ by default, causing Ruby's `rbconfig` to record `CXX = false`.

**Fix:** rebuild Ruby with the C++ include path visible to `./configure`:

```bash
CPLUS_INCLUDE_PATH=/Library/Developer/CommandLineTools/SDKs/MacOSX26.1.sdk/usr/include/c++/v1 \
  ruby-install ruby 3.3.11 -- CXX=clang++
```

Then add the following to `~/.zshrc` so the path is available in every session:

```bash
export CPLUS_INCLUDE_PATH=/Library/Developer/CommandLineTools/SDKs/MacOSX26.1.sdk/usr/include/c++/v1
```

Open a new terminal, `cd` into the project (chruby will auto-activate Ruby 3.3.11 via `.ruby-version`), and run `bundle install`.

To test the theme, run `bundle exec rake preview` and open your browser at `http://localhost:4000/test/`. This starts a Jekyll server using content in the `test/` directory. As modifications are made to the theme and test site, it will regenerate and you should see the changes in the browser after a refresh.
