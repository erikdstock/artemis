# Artemis [![Build Status](https://travis-ci.org/yuki24/artemis.svg?branch=master)](https://travis-ci.org/yuki24/artemis)

 * **Convention over Configuration**: You'll never have to make trivial decisions or spend time on boring setup. Start
  making a GraphQL request in literally 30sec.
 * **Performant by default**: You can't do wrong when it comes to performance. All GraphQL files are pre-loaded only
  once in production and it'll never affect runtime performance. Comes with options that enable persistent connections
   and even HTTP/2.0, the next-gen high-performance protocol.

<img width="24" height="24" src="https://avatars1.githubusercontent.com/u/541332?s=48&amp;v=4"> Battled-tested at [Artsy](https://www.artsy.net)

## Getting started

Add this line to your application's Gemfile:

```ruby
gem 'artemis'
```

And then execute:

    $ bundle

Once you run `bundle install` on your Rails app, run the install command:


```sh
$ rails g artemis:install artsy https://metaphysics-production.artsy.net/
```

You could also use the `--authorization` option to assign a token so the installer can download the GraphQL schema:

```sh
$ rails g artemis:install github https://api.github.com/graphql --authorization 'token ...'
```

### Generating your first query

Artemis comes with a query generator. For exmaple, you could use the query generator to generate a query stub for `artist`:

```sh
$ rails g artemis:query artist
```

Then this will generate:

```graphql
# app/operations/artist.graphql
query($id: String!) {
  artist(id: $id) {
    # Add fields here...
  }
}
```

Then you could the class method that has the matching name `artist`:

```ruby
Artsy.artist(id: "pablo-picasso")
# => makes a GraphQL query that's in app/operations/artist.graphql
```

You can also specify a file name:

```sh
$ rails g artemis:query artist artist_details_on_artwork
# => generates app/operations/artist_details_on_artwork.graphql
```

Then you can make a query in `artist_details_on_artwork.graphql` with:

```ruby
Artsy.artist_details_on_artwork(id: "pablo-picasso")
```

## The convention

Artemis assumes that the files related to GraphQL are organized in a certain way. For example, a service that talks to Artsy's GraphQL API could have the following structure:

```
├──app/operations
│   ├── artsy
│   │   ├── _artist_fragment.graphql
│   │   ├── artwork.graphql
│   │   ├── artist.graphql
│   │   └── artists.graphql
│   └── artsy.rb
├──config/graphql.yml
└──vendor/graphql/schema/artsy.json
```

## Callbacks

Youcan use the `before_execute` callback to intercept outgoing requests and the `after_execute` callback to observe the
response. A common operation that's done in the `before_execute` hook is assigning a token to the header:

```ruby
class Artsy < Artemis::Client
  before_execute do |document, operation_name, variables, context|
    context[:headers] = {
      Authorization: "token ..."
    }
  end
end
```

Here the `:headers` key is a special context type. The hash object assigned to the `context[:headers]` will be sent as
the HTTP headers of the request.

Another common thing when receiving a response is to check if there's any error in the response and throw and error
accordingly:

```ruby
class Artsy < Artemis::Client
  after_execute do |data, errors, extensions|
    raise "GraphQL error: #{errors.to_json}" if errors.present?
  end
end
```

## Configuration

You can configure the GraphQL client using the following options. Those configurations are found in the
`config/graphql.yml`.

| Name          | Required? | Default     | Description |
| ------------- | --------- | ------------| ----------- |
| `adapter`     | No        | `:net_http` | The underlying client library that actually makes an HTTP request. See Adapters for available options.
| `pool_size`   | No        | 25          | The number of keep-alive connections. The `:net_http` adapter will ignore this option.
| `schema_path` | No        | See above   | The path to the GrapQL schema. Setting an empty value to this will force the client to download the schema upon the first request.
| `timeout`     | No        | 10          | HTTP timeout set for the adapter in seconds. This will be set to both `read_timeout` and `write_timeout` and there is no way to configure them with a different value as of writing (PRs welcome!)
| `url`         | Yes       | N/A         | The URL for the GraphQL endpoint.

### Adapters

There are four adapter options available. Choose the adapter that best fits on your use case.

| Adapter                | Protocol                 | Keep-alive  | Performance | Dependencies |
| ---------------------- | ------------------------ | ----------- | ----------- | ------------ |
| `:curb`                | HTTP/1.1, **HTTP/2.0**   | **Yes**     | **Fastest** | [`curb 0.9.6+`][curb]<br>[`libcurl 7.64.0+`][curl]<br>[`nghttp2 1.0.0+`][nghttp]
| `:net_http` (default)  | HTTP/1.1 only            | No          | Slow        | **None**
| `:net_http_persistent` | HTTP/1.1 only            | **Yes**     | **Fast**    | [`net-http-persistent 3.0.0+`][nhp]
| `:test`                | N/A (See Testing)

## Rake tasks

Artemis also adds a useful `rake graphql:schema:update` rake task that downloads the GraphQL schema using the
`Introspection` query.

| Task Name                    | Options   | Description |
| ---------------------------- | --------- | ------------|
| `graphql:schema:update` | `SERVICE`: Service name the schema is downloaded from<br>`AUTHORIZATION`: HTTTP `Authroization` header value used to download the schema| Downloads and saves the GraphQL schema

## Testing

**The testing support is incomplete, but there are some examples [available in Artemis' client spec](https://github.com/yuki24/artemis/blob/74095f3acb050e87251439aed5f8b17778ffdd06/spec/client_spec.rb#L36-L54).**

## Development

After checking out the repo, run `bin/setup` to install dependencies. Then, run `rake test` to run the tests. You can also run `bin/console` for an interactive prompt that will allow you to experiment.

To install this gem onto your local machine, run `bundle exec rake install`. To release a new version, update the version number in `version.rb`, and then run `bundle exec rake release`, which will create a git tag for the version, push git commits and tags, and push the `.gem` file to [rubygems.org](https://rubygems.org).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/yuki24/artemis. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.

## License

The gem is available as open source under the terms of the [MIT License](https://opensource.org/licenses/MIT).

## Code of Conduct

Everyone interacting in the Artemis project’s codebases, issue trackers, chat rooms and mailing lists is expected to follow the [code of conduct](https://github.com/[USERNAME]/artemis/blob/master/CODE_OF_CONDUCT.md).

[curb]: https://rubygems.org/gems/curb
[curl]: https://curl.haxx.se/docs/http2.html
[nghttp]: https://nghttp2.org/
[nhp]: https://rubygems.org/gems/net-http-persistent