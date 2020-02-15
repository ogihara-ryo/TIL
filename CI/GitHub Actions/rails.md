Ruby 2.7 と PostgreSQL で Rubocop と RSpec を通す。

また、`actions/setup-ruby` よりも `ruby/setup-ruby` を使った方が良いとのこと。  
参考: https://mstshiwasaki.hatenablog.com/entry/2020/02/08/130844

```yml
name: Rubocop & RSpec

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    services:
      postgres:
        image: postgres:12
        ports:
          - 5432:5432
        env:
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
        options: --health-cmd pg_isready --health-interval 10s --health-timeout 5s --health-retries 5

    steps:
      - uses: actions/checkout@v1

      - uses: actions/cache@v1
        with:
          path: vendor/bundle
          key: bundle-${{ hashFiles('**/Gemfile.lock') }}

      - name: Set up Ruby 2.7
        uses: actions/setup-ruby@v1
        with:
          ruby-version: 2.7.0

      - uses: borales/actions-yarn@v2.0.0
        with:
          cmd: install

      - name: install bundler2
        run: gem install bundler

      - name: Install dependent libralies
        run: sudo apt-get install libpq-dev

      - name: Bundle install
        run: bundle install --jobs 4 --retry 3 --path vendor/bundle

      - name: Setup Database
        run: |
          cp config/database.yml.github-actions config/database.yml
          bundle exec rake db:create
          bundle exec rake db:schema:load
        env:
          RAILS_ENV: test
          RAILS_DATABASE_HOST: postgres
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres

      - name: Rubocop
        run: bundle exec rubocop

      - name: RSpec
        run: COVERAGE=true bundle exec rspec  --require rails_helper
        env:
          RAILS_ENV: test
          POSTGRES_USER: postgres
          POSTGRES_PASSWORD: postgres
```
