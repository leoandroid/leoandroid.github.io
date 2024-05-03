# Leo's Blog

## Dependence applications version

| Application | Version |
| :---------: | :-----: |
| Mac         | 14.4.1 (23E224)|
| Ruby        | 3.3.1   |
| gem         | 3.5.9   |
| jekyll      | 4.3.3   |
| Bundler     | 2.5.9   |

## Run

```shell
# install plugin and depences
bundle install
# Kill running jekyll server
ps aux |grep jekyll |awk '{print $2}' | xargs kill -9
# run jekyll liveload server
bundle exec jekyll serve --livereload
# open chrome and load the local jekyll
open -n -a "Google Chrome" "http://localhost:4000/"
```

## Issue solution
If the ruby install by homebrew has confict with the mac's ruby, add the below code to `~/.zshrc`,
then run `source ~/.zshrc`.

```shell
if [ -d "/opt/homebrew/opt/ruby/bin" ]; then
 export PATH=/opt/homebrew/opt/ruby/bin:$PATH
 export PATH=`gem environment gemdir`/bin:$PATH
fi 
```