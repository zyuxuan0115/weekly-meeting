## weekly meeting updates of code layout optimization at runtime
To make it work on your Mac
```bash
> brew install chruby ruby-install xz
> ruby-install ruby
> echo "source $(brew --prefix)/opt/chruby/share/chruby/chruby.sh" >> ~/.zshrc
> echo "source $(brew --prefix)/opt/chruby/share/chruby/auto.sh" >> ~/.zshrc
> echo "chruby ruby-3.1.2" >> ~/.zshrc # run 'chruby' to see actual version
> sudo gem install jekyll
> git clone git@github.com:zyuxuan0115/weekly-meeting.git
> cd weekly-meeting
> bundle exec jekyll serve
```
Then browse to http://localhost:4000

Or, you can directly open files in [_posts](https://github.com/zyuxuan0115/weekly-meeting/tree/main/_posts) directory.
