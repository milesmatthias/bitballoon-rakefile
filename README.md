bitballoon-rakefile
===================

A rakefile to work with bitballoon. Put your keys in the config file, and you're good to go.


usage
=====

1. put your client key and secret in `config/bitballoon.yaml`
2. edit the `BUILD_DIR` constant in the `Rakefile` if necessary (default is `build`)
3. `bundle install`
4. `bundle exec rake bitballoon:deploy`
5. profit.
